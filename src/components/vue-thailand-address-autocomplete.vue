<template>
  <div class="vth-addr-container" v-click-outside="onClickOutside">
    <div class="vth-addr-label" v-if="label">{{ label }}</div>
    <div class="vth-addr-input-container">
      <q-input type="text"
       outlined rounded
      v-model="currentValue"
      :label="placeholder"
      autocomplete="disabled"
      ref="input"
      @focus="hasFocus = true"
      @blur="hasFocus = false"
      @keydown.up="pressArrow('up')"
      @keydown.down="pressArrow('down')"
      @keydown.enter="pressEnter()"/>
      <div v-if="resultsFromSearch.length && isOpenListContainer"
      ref="dropdown"
      class="vth-addr-list-container"
      :style="{'top': findListContainerPosition() }">
        <div class="vth-addr-list"
        :class="{ 'vth-addr-list-on-focused': itemOnFocus === index }"
        :style="{
          'background-color': itemOnFocus === index ? currentColor : '#fff'
        }"
        v-for="(item, index) in resultsFromSearch"
        :key="index"
        @mouseover="itemOnFocus = index"
        @mouseout="itemOnFocus = -1"
        @click="clickSelectItem(item)">
          <div class="vth-addr-box-item-top">
            <span class="item-first" :class="{ 'vth-addr-box-item-top-focused': itemOnFocus === index && currentColor !== '#f5f5f5' }">
              {{ itemFirst(item) }}
            </span>
            <div class="vth-addr-float-right">
              <span class="vth-addr-item-second" :class="{ 'vth-addr-box-item-top-focused': itemOnFocus === index && currentColor !== '#f5f5f5' }">
                {{ itemSecond(item) }}
              </span>
              <span class="vth-addr-item-third" :class="{ 'vth-addr-box-item-top-focused': itemOnFocus === index && currentColor !== '#f5f5f5' }">
                {{ itemThird(item) }}
              </span>
            </div>
          </div>
          <div class="vth-addr-box-item-bottom">
            <span class="vth-addr-item-first vth-addr-font-weight-bold" :style="{'color': itemOnFocus === index && currentColor !== '#f5f5f5' ? '#fff' : '#000'}">
              {{ itemFourth(item) }}
            </span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { searchAddressByDistrict, searchAddressByAmphoe, searchAddressByProvince, searchAddressByZipcode } from 'thai-address-database'
import vClickOutside from 'v-click-outside'
/*eslint-disable */

export default {
  name: 'VueThailandAddressAutocomplete',
  props: {
    value: {
      type: [String, Number]
    },
    type: {
      type: String
    },
    label: {
      type: String
    },
    placeholder: {
      type: String
    },
    color: {
      type: String
    },
    size: {
      type: String,
      default: 'default'
    }
  },
  data () {
    return {
      currentValue: this.value,
      currentColor: this.color || '#f5f5f5',
      itemOnFocus: 0,
      isOpenListContainer: false,
      hasFocus: false
    }
  },
  directives: {
    clickOutside: vClickOutside.directive
  },
  computed: {
    resultsFromSearch () {
      if (this.type) {
        if (this.type === 'district') return this.resultsFromSearchByDistrict
        else if (this.type === 'amphoe') return this.resultsFromSearchByAmphoe
        else if (this.type === 'province') return this.resultsFromSearchByProvince
        else if (this.type === 'zipcode') return this.resultsFromSearchByZipcode
      } else {
        this._errorLog('type is undefined.')
        return []
      }
    },
    resultsFromSearchByDistrict () {
      return searchAddressByDistrict(this.currentValue)
    },
    resultsFromSearchByAmphoe () {
      return searchAddressByAmphoe(this.currentValue)
    },
    resultsFromSearchByProvince () {
      return searchAddressByProvince(this.currentValue)
    },
    resultsFromSearchByZipcode () {
      return searchAddressByZipcode(this.currentValue)
    }
  },
  watch: {
    /**
     * - Update v-model of user.
     * - Set ????????? List Container ??????????????????????????? currentValue ????????????????????????????????????????????????
     * - Set itemOnFocus = first item.
     */
    currentValue (value) {
      this.$emit('input', this.currentValue)
      if (this.hasFocus) {
        this.isOpenListContainer = true
      }
      this.itemOnFocus = 0
    },
    /**
     * - ????????? v-model ???????????????????????????????????????????????? Component
     * - ??????????????????????????????????????? Internal value ????????????
     * - ????????????????????? Dropdown
     */
    value (value) {
      if (value !== this.currentValue) {
        this.currentValue = value
        this.$nextTick(() => this.isOpenListContainer = false)
      }
    },
    /**
     * - When color is changed: set internal value.
     */
    color (val) {
      this.currentColor = val
    }
  },
  methods: {
    onClickOutside (event) {
      this.isOpenListContainer = false
    },
    /**
    * Arrows keys listener.
    */
    pressArrow (direction) {
      if (direction === 'up') {
        this.setInputCursorToLastChar()
        this.itemOnFocus = this.itemOnFocus > 0 ? this.itemOnFocus - 1 : 0
      } else {
        this.itemOnFocus = this.itemOnFocus < this.resultsFromSearch.length - 1 ? this.itemOnFocus + 1 : this.resultsFromSearch.length - 1
      }
      this.moveScrollOfListContainer()
    },
    /**
    * Enter button listener.
    */
    pressEnter () {
      this.setSelectedValue(this.resultsFromSearch[this.itemOnFocus])
    },
    /**
    * ?????????????????? scroll bar ????????? Item ????????????????????????
    */
    moveScrollOfListContainer () {
      const list = this.$refs.dropdown
      const element = list.querySelectorAll('.list')[this.itemOnFocus]
      if (!element) return
      const visMin = list.scrollTop
      const visMax = list.scrollTop + list.clientHeight - element.clientHeight
      if (element.offsetTop < visMin) {
        list.scrollTop = element.offsetTop
      } else if (element.offsetTop >= visMax) {
        list.scrollTop = (
          element.offsetTop -
          list.clientHeight +
          element.clientHeight
        )
      }
    },
    /**
    * Event on click item.
    */
    clickSelectItem (address) {
      this.setSelectedValue(address)
    },
     /**
     * - emit ?????????????????????????????????????????????????????????????????????????????????????????? user
     * - Set currentValue = ??????????????????????????? user ????????????????????????
     * - ???????????? List Container
     */
    setSelectedValue (address) {
      this.$emit('select', address)
      this.type ? this.currentValue = address[this.type] : this._errorLog('type is undefined.')
      this.$nextTick(() => this.isOpenListContainer = false)
    },
     /**
     * - ??????????????? top ????????? Dropwon Container
     */
    findListContainerPosition () {
      let top = '36px'
      if (this.size === 'small') {
        top = '27px'
      } else if (this.size === 'medium') {
        top = '45px'
      } else if (this.size === 'large') {
        top = '54px'
      }
      return top
    },
    /**
     * - ??????????????? Cursor ????????? input ?????????????????????????????????????????????
     */
    setInputCursorToLastChar () {
      const len = this.currentValue.length * 2
      setTimeout(() => { this.$refs.input.setSelectionRange(len, len) }, 1)
    },
    /**
     * - ????????????????????????????????????????????????
     */
    itemFirst (address) {
      return this.type === 'district' ? address['amphoe'] : address['district']
    },
    /**
     * - ?????????????????????????????????????????????????????????
     */
    itemSecond (address) {
      return this.type === 'amphoe' || this.type === 'district' ? address['province'] : address['amphoe']
    },
    /**
     * - ?????????????????????????????????????????????
     */
    itemThird (address) {
      return this.type === 'province' || this.type === 'amphoe' || this.type === 'district' ? address['zipcode'] : address['province']
    },
    /**
     * - ??????????????????????????????????????????????????????
     */
    itemFourth (address) {
      return address[this.type]
    },
    /**
     * - Error Log
     */
    _errorLog (text) {
      console.error(`[ERROR] vue-thailand-address-autocomplete : ${text}`)
    }
  }
}
</script>

<style scoped>
.q-field--outlined {
  padding: 0 5px;
}
</style>
