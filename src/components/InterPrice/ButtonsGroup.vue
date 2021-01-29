<template>
  <div class="flex justify-center rounded-lg text-lg" role="group">
    <button
        v-for="(value, index) in Object.keys(values)"
        :key="`button-${value}`"
        :class="determineClass(value, index)"
        @click="handleClick(value)"
    >{{ value }}
    </button>
  </div>
</template>

<script>
const TAILWIND_ACTIVE = 'bg-blue-500 text-white'
const TAILWIND_IDLE = 'bg-white text-blue-500'
const TAILWIND_ALONE = 'border border-blue-500 rounded-l-lg rounded-r-lg px-4 py-2 mx-0 outline-none focus:outline-none'
const TAILWIND_FIRST = 'border border-blue-500 rounded-l-lg px-4 py-2 mx-0 outline-none focus:outline-none'
const TAILWIND_MIDDLE = 'border border-l-0 border-blue-500 px-4 py-2 mx-0 outline-none focus:outline-none'
const TAILWIND_LAST = 'border border-l-0 border-blue-500 rounded-r-lg px-4 py-2 mx-0 outline-none focus:outline-none'

export default {
  name: 'ButtonsGroup',
  props: {
    values: {
      type: Object,
      required: true
    },
    multiple: {
      type: Boolean,
      default: false
    }
  },
  methods: {
    determineClass(value, index) {
      const prefix = (this.values[value] ? TAILWIND_ACTIVE : TAILWIND_IDLE) + ' '
      const buttonsCount = Object.keys(this.values).length
      if (buttonsCount < 2) {
        return prefix + TAILWIND_ALONE
      }
      switch (index) {
        case 0:
          return prefix + TAILWIND_FIRST
        case buttonsCount - 1:
          return prefix + TAILWIND_LAST
        default:
          return prefix + TAILWIND_MIDDLE
      }
    },
    handleClick(value) {
      let values = Object.assign({}, this.values)
      if (!this.multiple) {
        if (values[value]) {
          return // you can not deselect all if not multiple
        }
        values = Object.fromEntries(
            Object.keys(values).map(key => [key, false])
        )
      }
      values[value] = !values[value]
      this.$emit('change', values)
    }
  }

}
</script>

<style scoped>

</style>