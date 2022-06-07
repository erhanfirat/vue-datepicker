<template>
  <div class="input-wrapper" :class="{ focus: isFocus }">
    <div class="prefix">
      <slot name="prefix" />
    </div>
    <div class="input-el-wrapper" :style="inputStyle">
      <label :for="id">
        {{ displayedValue[0] }}
      </label>
      <input
        class="vue-input"
        ref="input0"
        :id="id"
        :value="value[0]"
        :placeholder="placeholder || '请选择开始'"
        :style="inputStyle"
        @focus="$emit('focus', 0)"
        readonly
      />
    </div>

    <div v-if="isRange" class="range-separator">
      {{ rangeSeparator || '至' }}
    </div>

    <div v-if="isRange" class="input-el-wrapper range" :style="inputStyle">
      <label :for="id + '-range-2'">
        {{ displayedValue[1] }}
      </label>
      <input
        class="vue-input"
        ref="input1"
        :id="id + '-range-2'"
        :value="value[1]"
        :placeholder="secondPlaceholder || '请选择结束'"
        :style="inputStyle"
        @focus="$emit('focus', 1)"
        readonly
      />
    </div>
    <div class="suffix" v-if="clearButton && canEdit">
      <slot name="suffix" />
      <div ref="clear" class="icon-clear" @click="$emit('clear')">
        <icon-del />
      </div>
    </div>
  </div>
</template>

<script>
import { DateTime } from 'luxon'
import IconDel from './IconDel'

export default {
  name: 'InputEl',
  props: {
    id: String,
    timeType: String,
    value: Array,
    placeholder: String,
    secondPlaceholder: String,
    inputStyle: Object,
    dayStr: Array,
    firstDayOfWeek: Number,
    rangeSeparator: String,
    isFocus: Boolean,
    isRange: Boolean,
    clearButton: Boolean,
    canEdit: Boolean,
    hour12: Boolean,
    dataType: String,
  },
  components: { IconDel },
  computed: {
    displayedValue() {
      if (this.isRange)
        return [
          this.formatValue(this.value[0]),
          this.formatValue(this.value[1]),
        ]
      return [this.formatValue(this.value[0])]
    },
    localeSettingsAndFormat() {
      let dataFormat = ''
      let localeSettings = {}
      if (this.dataType === 'date' || this.dataType === 'datetime') {
        dataFormat = 'yyyy-MM-dd'
        localeSettings = { year: 'numeric', month: 'numeric', day: 'numeric' }
      }

      if (this.dataType === 'time' || this.dataType === 'datetime') {
        dataFormat += this.dataType === 'datetime' ? ' ' : ''
        if (this.timeType === 'hour') {
          localeSettings = {
            ...localeSettings,
            hour: '2-digit',
            hour12: this.hour12,
          }
          dataFormat += 'hh'
        } else if (this.timeType === 'minute') {
          localeSettings = {
            ...localeSettings,
            hour: '2-digit',
            minute: '2-digit',
            hour12: this.hour12,
          }
          dataFormat += 'hh:mm'
        } else if (this.timeType === 'second') {
          localeSettings = {
            ...localeSettings,
            hour: '2-digit',
            minute: '2-digit',
            second: '2-digit',
            hour12: this.hour12,
          }
          dataFormat += 'hh:mm:ss'
        }
      }
      return { dataFormat, localeSettings }
    },
  },
  methods: {
    formatValue(value) {
      let result = ''
      if (value) {
        const { dataFormat, localeSettings } = this.localeSettingsAndFormat
        result = DateTime.fromFormat(value, dataFormat).toLocaleString(
          localeSettings,
        )

        if (result.indexOf('Invalid') > -1) result = value
      }
      return result
    },
  },
}
</script>
