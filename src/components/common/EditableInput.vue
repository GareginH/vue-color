<template>
  <div class="vc-editable-input">
    <input
      :aria-labelledby="labelId"
      class="vc-input__input"
      :class="{'vc-input__rgba': inputType === $options.TYPE_RGBA}"
      v-model="val"
      @keydown="handleKeyDown"
      @input="update"
      ref="input"
    >
    <span v-if="inputType === $options.TYPE_RGBA" :for="label" class="vc-input__label" :id="labelId">{{labelSpanText}}</span>
    <span v-if="inputType === $options.TYPE_RGBA" class="vc-input__desc">{{desc}}</span>
  </div>
</template>

<script>
export default {
  TYPE_RGBA: 1,
  name: 'editableInput',
  props: {
    label: String,
    labelText: String,
    desc: String,
    value: [String, Number],
    max: Number,
    min: Number,
    inputType: {
      type: [String, Number],
      default: 0
    },
    arrowOffset: {
      type: Number,
      default: 1
    }
  },
  computed: {
    val: {
      get () {
        return this.value
      },
      set (v) {
        // TODO: min
        if (!(this.max === undefined) && +v > this.max) {
          this.$refs.input.value = this.max
        } else {
          return v
        }
      }
    },
    labelId () {
      return `input__label__${this.label}__${Math.random().toString().slice(2, 5)}`
    },
    labelSpanText () {
      return this.labelText || this.label
    }
  },
  methods: {
    update (e) {
      this.handleChange(e.target.value)
    },
    handleChange (newVal) {
      let data = {}
      data[this.label] = newVal
      if (data.hex === undefined && data['#'] === undefined) {
        this.$emit('change', data)
      } else if (newVal.length > 5) {
        this.$emit('change', data)
      }
    },
    // **** unused
    // handleBlur (e) {
    //   console.log(e)
    // },
    handleKeyDown (e) {
      let val = this.val
      let number = Number(val)

      if (number) {
        let amount = this.arrowOffset || 1

        // Up
        if (e.keyCode === 38) {
          val = number + amount
          this.handleChange(val)
          e.preventDefault()
        }

        // Down
        if (e.keyCode === 40) {
          val = number - amount
          this.handleChange(val)
          e.preventDefault()
        }
      }
    }
    // **** unused
    // handleDrag (e) {
    //   console.log(e)
    // },
    // handleMouseDown (e) {
    //   console.log(e)
    // }
  }
}
</script>

<style>
.vc-editable-input {
  position: relative;
}
.vc-input__input {
  padding: 0;
  border: 0;
  outline: none;
}
.vc-input__rgba {
  font-size: 14px!important;
  padding: 5px 8px!important;
  height: unset!important;
  text-align: center!important;
}
.vc-input__label {
  text-transform: capitalize;
}
</style>
