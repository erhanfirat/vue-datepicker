<template>
  <div class="picker-items time">
    <div class="row-item h">
      <span
        v-for="(item, i) in $timeStr"
        class="item second"
        :style="pickerItemStyle"
        :key="i"
        >{{ item }}</span
      >
    </div>
    <scrollbar
      v-for="(rowItems, i) in pickerItems"
      class="row-item line"
      ref="scrollbar"
      :style="pickerItemStyle"
      :key="i"
      :isMobile="$scrollbarProps.isMobile"
      :maxHeight="$scrollbarProps.maxHeight"
      :marginToWrap="$scrollbarProps.marginToWrap"
    >
      <template v-for="(item, j) in rowItems">
        <span
          v-if="checkTimeListItemVisible(i, item.value)"
          class="item"
          :class="{
            disabled: !item.canBeChose,
            [i]: true,
            selected: isSelected(item, i),
          }"
          :key="i + '' + j"
          @click="chose(item, i)"
        >
          <template v-if="i === 'hour' && hour12">
            {{ formatHour12(item.value) }}
            <span class="meridiem">{{ meridiemOfHour(item.value) }}</span>
          </template>
          <template v-else>
            {{ item.value }}
          </template>
        </span>
      </template>
    </scrollbar>
  </div>
</template>

<script>
import VueScrollbar from 'vue-scrollbar-live'
import {
  fillTo,
  getHour,
  getMinute,
  getSecond,
} from '@livelybone/date-generator'
import { createNowTimeObj, formatTimeObj, timeCompare } from './utils'

export default {
  name: 'Time',
  props: {
    selectedTime: Object,
    type: String,
    scrollbarProps: Object,
    minTime: Object,
    maxTime: Object,
    timeStr: Array,
    hourStep: Number,
    minuteStep: Number,
    secondStep: Number,
    hour12: Boolean,
  },
  data() {
    return {
      timeObj: formatTimeObj(createNowTimeObj()),
    }
  },
  computed: {
    myType() {
      return /^(hour|minute|second)$/.test(this.type) ? this.type : 'second'
    },
    $scrollbarProps() {
      return { maxHeight: 200, ...this.scrollbarProps }
    },
    $timeStr() {
      const arr =
        !this.timeStr ||
        this.timeStr.length < 3 ||
        this.timeStr.some(day => typeof day !== 'string')
          ? ['时', '分', '秒']
          : this.timeStr.slice(0, 3)
      if (this.myType === 'hour') return arr.slice(0, 1)
      if (this.myType === 'minute') return arr.slice(0, 2)
      return arr.slice(0, 3)
    },
    hours() {
      return getHour({
        min: this.minTime.hour,
        max: this.maxTime.hour,
      })
    },
    minutes() {
      return getMinute({
        min:
          +this.timeObj.hour === +this.minTime.hour ? this.minTime.minute : 0,
        max:
          +this.timeObj.hour === +this.maxTime.hour ? this.maxTime.minute : 59,
      })
    },
    seconds() {
      return getSecond({
        min:
          +this.timeObj.hour === +this.minTime.hour &&
          +this.timeObj.minute === +this.minTime.minute
            ? this.minTime.second
            : 0,
        max:
          +this.timeObj.hour === +this.maxTime.hour &&
          +this.timeObj.minute === +this.maxTime.minute
            ? this.maxTime.second
            : 59,
      })
    },
    pickerItems() {
      const obj = { hour: this.hours }
      if (this.myType === 'hour') return obj
      obj.minute = this.minutes
      if (this.myType === 'minute') return obj
      obj.second = this.seconds
      return obj
    },
    pickerItemStyle() {
      if (this.myType === 'hour') return { width: '100%' }
      if (this.myType === 'minute') return { width: 'calc(100% / 2)' }
      return { width: 'calc(99% / 3)' }
    },
  },
  watch: {
    'timeObj.hour': {
      handler() {
        if (!timeCompare(this.timeObj, this.minTime, 1, 'minute'))
          this.timeObj.minute = fillTo(2, this.minTime.minute)
        else if (!timeCompare(this.timeObj, this.maxTime, -1, 'minute'))
          this.timeObj.minute = fillTo(2, this.maxTime.minute)
        else if (!timeCompare(this.timeObj, this.minTime))
          this.timeObj.second = fillTo(2, this.minTime.second)
        else if (!timeCompare(this.timeObj, this.maxTime, -1))
          this.timeObj.second = fillTo(2, this.maxTime.second)
        else this.$emit('chose', { type: 'hour', ...this.timeObj })
        this.$forceUpdate()
      },
    },
    'timeObj.minute': {
      handler() {
        if (!timeCompare(this.timeObj, this.minTime))
          this.timeObj.second = fillTo(2, this.minTime.second)
        else if (!timeCompare(this.timeObj, this.maxTime, -1))
          this.timeObj.second = fillTo(2, this.maxTime.second)
        else this.$emit('chose', { type: 'minute', ...this.timeObj })
        this.$forceUpdate()
      },
    },
    'timeObj.second': {
      handler() {
        this.$emit('chose', { type: 'second', ...this.timeObj })
      },
    },
  },
  methods: {
    chose(item, type = 'second') {
      if (item.canBeChose) {
        if (type === 'second') {
          this.timeObj.second = item.value
        } else if (type === 'minute') {
          this.timeObj.minute = item.value
        } else if (type === 'hour') {
          this.timeObj.hour = item.value
        }
        this.timeObj = { ...this.timeObj }
      }
    },
    isSelected(item, type) {
      return this.selectedTime && this.selectedTime[type] === item.value
    },
    checkTimeListItemVisible(type, value) {
      return parseInt(value, 10) % this[`${type}Step`] === 0
    },
    formatHour12(hour) {
      if (parseInt(hour, 10) === 12 || parseInt(hour, 10) === 0) return 12
      return hour % 12
    },
    meridiemOfHour(hour) {
      return hour >= 12 ? 'PM' : 'AM'
    },
  },
  components: { scrollbar: VueScrollbar },
}
</script>
