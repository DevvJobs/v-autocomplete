<template lang="html">
  <div class="v-autocomplete">
    <div class="v-autocomplete-input-group" :class="{'v-autocomplete-selected': value}">
      <input type="search" v-model="searchText" v-bind="inputAttrs" 
            :class="inputAttrs.class || inputClass"
            :placeholder="inputAttrs.placeholder || placeholder"
            :disabled="inputAttrs.disabled || disabled"
            @blur="blur" @focus="focus" @input="inputChange"
            @keyup.enter="keyEnter" @keydown.tab="keyEnter" 
            @keydown.up="keyUp" @keydown.down="keyDown">
      <svg class="v-autocomplete-icon"  xmlns="http://www.w3.org/2000/svg" viewBox="0 0 13.09 7.71">
        <path d="M6.3 7.69A.73.73 0 0 0 6 7.6l-.17-.09L.34 2.06A1.13 1.13 0 0 1 0 1.23 1.26 1.26 0 0 1 .34.36 1.15 1.15 0 0 1 1.18 0 1.21 1.21 0 0 1 2 .36l4.54 4.47L11.05.36A1.16 1.16 0 0 1 11.9 0a1.14 1.14 0 0 1 .84.35 1.15 1.15 0 0 1 .35.85 1.2 1.2 0 0 1-.34.86L7.24 7.49l-.12.07-.12.03-.23.08h-.31z" data-name="COUNTRY EDIT"/>
      </svg>
    </div>
    <div class="v-autocomplete-list-holder" v-if="show">
      <div class="v-autocomplete-list">
        <div class="v-autocomplete-list-item" v-for="item, i in internalItems" @click="onClickItem(item)"
             :class="{'v-autocomplete-item-active': i === cursor}" @mouseover="cursor = i">
          <div :is="componentItem" :item="item" :searchText="searchText"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Item from './Item.vue'
import utils from './utils.js'

export default {
  name: 'v-autocomplete',
  props: {
    componentItem: { default: () => Item },
    minLen: { type: Number, default: utils.minLen },
    wait: { type: Number, default: utils.wait },
    value: null,
    getLabel: {
      type: Function,
      default: item => item
    },
    items: Array,
    autoSelectOneItem: { type: Boolean, default: true },
    placeholder: String,
    inputClass: {type: String, default: 'v-autocomplete-input'},
    disabled: {type: Boolean, default: false},
    inputAttrs: {type: Object, default: () => {return {}}},
    keepOpen: {type: Boolean, default: false},
    scrollItemsIntoView: {type: Boolean, default: true},
    scrollOptions: {
      type: [Boolean, Object],
      default: {
        behavior: "smooth", block: "center", inline: "center"
      }
    }
  },
  data () {
    return {
      searchText: '',
      showList: false,
      cursor: -1,
      internalItems: this.items || []
    }
  },
  computed: {
    hasItems () {
      return !!this.internalItems.length
    },
    show () {
      return (this.showList && this.hasItems) || this.keepOpen
    }
  },
  methods: {
    inputChange () {
      this.showList = true
      this.cursor = -1
      this.onSelectItem(null, 'inputChange')
      utils.callUpdateItems(this.searchText, this.updateItems)
      this.$emit('change', this.searchText)
    },

    updateItems () {
      this.$emit('update-items', this.searchText)
    },

    focus () {
      this.$emit('focus', this.searchText)
      this.showList = true
    },

    blur () {
      this.$emit('blur', this.searchText)
      setTimeout( () => this.showList = false, 200)
    },

    onClickItem(item) {
      this.onSelectItem(item)
      this.$emit('item-clicked', item)
    },

    onSelectItem (item) {
      if (item) {
        this.internalItems = [item]
        this.searchText = this.getLabel(item)
        this.$emit('item-selected', item)
      } else {
        this.setItems(this.items)
      }
      this.$emit('input', item)
    },

    setItems (items) {
      this.internalItems = items || []
    },

    isSelectedValue (value) {
      return 1 == this.internalItems.length && value == this.internalItems[0]
    },

    keyUp (e) {
      if (this.cursor > -1) {
        this.cursor--
        this.itemView(this.$el.getElementsByClassName('v-autocomplete-list-item')[this.cursor])
      }
    },

    keyDown (e) {
      if (this.cursor < this.internalItems.length) {
        this.cursor++
        this.itemView(this.$el.getElementsByClassName('v-autocomplete-list-item')[this.cursor])
      }
    },

    itemView (item) {
      if (item && this.scrollItemsIntoView && item.scrollIntoView) {
        item.scrollIntoView(this.scrollOptions);
      }
    },

    keyEnter (e) {
      if (this.showList && this.internalItems[this.cursor]) {
        this.onSelectItem(this.internalItems[this.cursor])
        this.showList = false
      }
    },

  },
  created () {
    utils.minLen = this.minLen
    utils.wait = this.wait
    this.onSelectItem(this.value)
  },
  watch: {
    items (newValue) {
      this.setItems(newValue)
      let item = utils.findItem(this.items, this.searchText, this.autoSelectOneItem)
      if (item) {
        this.onSelectItem(item)
        this.showList = false
      }
    },
    value (newValue) {
      if (!this.isSelectedValue(newValue) ) {
        this.onSelectItem(newValue)
        this.searchText = this.getLabel(newValue)
      }
    }
  }
}
</script>

<style>
  .v-autocomplete-icon {
    position: absolute;
    top: 50%;
    right: 10px;
    transform: translateY(-50%);
    width: 13px;
    height: 7px;
    fill: #cfd8dc;
  }
  .v-autocomplete {
    position: relative;
  }
  .v-autocomplete .v-autocomplete-list {
    position: absolute;
  }
  .v-autocomplete .v-autocomplete-list .v-autocomplete-list-item {
    cursor: pointer;
  }
  .v-autocomplete .v-autocomplete-list .v-autocomplete-list-item.v-autocomplete-item-active {
    background-color: #f3f6fa;
  }
</style>
