<template>
  <div class="input-wrapper" :class="{ focus: isFocus }">
    <div class="prefix">
      <slot name="prefix" />
    </div>
    <input
      class="vue-input"
      ref="input0"
      :id="id"
      :value="value[0]"
      :placeholder="placeholder || '请选择开始'"
      :style="inputStyle"
      @keyup.enter="$emit('inputEnter', $event)"
      @focus="$emit('focus', 0)"
      readonly
    />
    <div v-if="isRange" class="range-separator">
      {{ rangeSeparator || '至' }}
    </div>
    <input
      v-if="isRange"
      class="vue-input"
      ref="input1"
      :value="value[1]"
      :placeholder="secondPlaceholder || '请选择结束'"
      :style="inputStyle"
      @keyup.enter="$emit('inputEnter', $event)"
      @focus="$emit('focus', 1)"
      readonly
    />
    <div class="suffix" v-if="clearButton && canEdit">
      <slot name="suffix" />
      <div ref="clear" class="icon-clear" @click="$emit('clear')">
        <icon-del />
      </div>
    </div>
  </div>
</template>

<script>
import IconDel from './IconDel'

export default {
  name: 'InputEl',
  props: {
    id: String,
    type: String,
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
  },
  components: { IconDel },
}
</script>
