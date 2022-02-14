<template>
  <div role="application" aria-label="Chrome color picker" :class="['vc-chrome', disableAlpha ? 'vc-chrome__disable-alpha' : '']">
    <div class="vc-chrome-saturation-wrap">
      <saturation v-model="colors" @change="childChange"></saturation>
    </div>
    <div class="vc-chrome-body">
      <div class="vc-chrome-controls">
        <div class="vc-chrome-color-wrap">
          <div :aria-label="`current color is ${colors.hex}`" class="vc-chrome-active-color" :style="{background: activeColor}"></div>
          <checkboard v-if="!disableAlpha"></checkboard>
        </div>

        <div class="vc-chrome-sliders">
          <div class="vc-chrome-hue-wrap">
            <hue v-model="colors" @change="childChange"></hue>
          </div>
          <div class="vc-chrome-alpha-wrap" v-if="!disableAlpha">
            <alpha v-model="colors" @change="childChange"></alpha>
          </div>
        </div>
      </div>

      <div class="vc-chrome-fields-wrap" v-if="!disableFields">
        <div class="vc-chrome-fields" v-show="fieldsIndex === 0">
          <!-- hex -->
          <div class="vc-chrome-field">
            <ed-in v-if="!hasAlpha" label="" :value="colors.hex" @change="inputChange"></ed-in>
            <ed-in v-if="hasAlpha" label="" :value="colors.hex8" @change="inputChange"></ed-in>
          </div>
        </div>
        <div class="vc-chrome-fields" v-show="fieldsIndex === 1">
          <!-- rgba -->
          <div class="vc-chrome-field">
            <ed-in :input-type="1" label="r" :value="colors.rgba.r" @change="inputChange"></ed-in>
          </div>
          <div class="vc-chrome-field">
            <ed-in :input-type="1" label="g" :value="colors.rgba.g" @change="inputChange"></ed-in>
          </div>
          <div class="vc-chrome-field">
            <ed-in :input-type="1" label="b" :value="colors.rgba.b" @change="inputChange"></ed-in>
          </div>
          <div class="vc-chrome-field" v-if="!disableAlpha">
            <ed-in :input-type="1" label="a" :value="colors.a" :arrow-offset="0.01" :max="1" @change="inputChange"></ed-in>
          </div>
        </div>
        <!-- btn -->
        <div class="vc-chrome-toggle-btn" role="button" aria-label="Change another color definition" @click="toggleViews">
          <span class="vc-chrome-toggle-text" v-show="fieldsIndex === 0">
            HEX
          </span>
          <span class="vc-chrome-toggle-text" v-show="fieldsIndex === 1">
            RGBA
          </span>
          <div class="vc-chrome-toggle-icon">
            <svg width="20" height="20" viewBox="0 0 20 20" fill="none" xmlns="http://www.w3.org/2000/svg">
              <path d="M5.83334 12.5L10 16.6667L14.1667 12.5" stroke="#4F4F4F" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
              <path d="M14.1667 7.49998L9.99999 3.33331L5.83333 7.49998" stroke="#4F4F4F" stroke-width="2" stroke-linecap="round" stroke-linejoin="round"/>
            </svg>
          </div>
        </div>
        <!-- btn -->
      </div>
    </div>
  </div>
</template>

<script>
import colorMixin from '../mixin/color'
import editableInput from './common/EditableInput.vue'
import saturation from './common/Saturation.vue'
import hue from './common/Hue.vue'
import alpha from './common/Alpha.vue'
import checkboard from './common/Checkboard.vue'

export default {
  name: 'Chrome',
  mixins: [colorMixin],
  props: {
    disableAlpha: {
      type: Boolean,
      default: false
    },
    disableFields: {
      type: Boolean,
      default: false
    }
  },
  components: {
    saturation,
    hue,
    alpha,
    'ed-in': editableInput,
    checkboard
  },
  data () {
    return {
      fieldsIndex: 0,
      highlight: false
    }
  },
  computed: {
    hsl () {
      const { h, s, l } = this.colors.hsl
      return {
        h: h.toFixed(),
        s: `${(s * 100).toFixed()}%`,
        l: `${(l * 100).toFixed()}%`
      }
    },
    activeColor () {
      const rgba = this.colors.rgba
      return 'rgba(' + [rgba.r, rgba.g, rgba.b, rgba.a].join(',') + ')'
    },
    hasAlpha () {
      return this.colors.a < 1
    }
  },
  methods: {
    childChange (data) {
      this.colorChange(data)
    },
    inputChange (data) {
      if (!data) {
        return
      }
      if (data.hex) {
        this.isValidHex(data.hex) && this.colorChange({
          hex: data.hex,
          source: 'hex'
        })
      } else if (data.r || data.g || data.b || data.a) {
        this.colorChange({
          r: data.r || this.colors.rgba.r,
          g: data.g || this.colors.rgba.g,
          b: data.b || this.colors.rgba.b,
          a: data.a || this.colors.rgba.a,
          source: 'rgba'
        })
      } else if (data.h || data.s || data.l) {
        const s = data.s ? (data.s.replace('%', '') / 100) : this.colors.hsl.s
        const l = data.l ? (data.l.replace('%', '') / 100) : this.colors.hsl.l

        this.colorChange({
          h: data.h || this.colors.hsl.h,
          s,
          l,
          source: 'hsl'
        })
      }
    },
    toggleViews () {
      if (this.fieldsIndex >= 1) {
        this.fieldsIndex = 0
        return
      }
      this.fieldsIndex ++
    }
  }
}
</script>

<style>
.vc-chrome {
  font-family: 'Lato';
  padding: 16px;
  background: #fff;
  box-shadow: 4px 4px 20px rgba(0, 0, 0, 0.08);
  box-sizing: initial;
  width: 285px;
  border-radius: 8px;
}
.vc-chrome-controls {
  display: flex;
}
.vc-chrome-color-wrap {
  position: relative;
  width: 56px;
}
.vc-chrome-active-color {
  position: relative;
  width: 48px;
  height: 48px;
  border-radius: 24px;
  overflow: hidden;
  z-index: 1;
  border: 1px solid rgba(0, 0, 0, 0.14);
}
.vc-chrome-color-wrap .vc-checkerboard {
  width: 48px;
  height: 48px;
  border-radius: 24px;
  background-size: auto;
}
.vc-chrome-sliders {
  flex: 1;
}
.vc-chrome-fields-wrap {
  padding-top: 16px;
  display: grid;
  grid-template-columns: 1fr 85px;
  gap: 8px;
}
.vc-chrome-fields {
  display: flex;
  margin-left: -6px;
  flex: 1;
}
.vc-chrome-field {
  width: 100%;
}
.vc-chrome-field:first-child {
  padding-left: 6px;
}
.vc-chrome-toggle-btn {
  display: flex;
  align-items: center;
  width: 85px;
  height: 46px;
  text-align: right;
  position: relative;
  background: #f3f3f3;
  border: 1px solid #f3f3f3;
  border-radius: 6px;
  cursor: pointer;
}
.vc-chrome-toggle-text {
  padding-left: 12px;
  font-size: 12px;
  line-height: 1.2;
  color: #999999;
}
.vc-chrome-toggle-icon {
  margin-right: -4px;
  margin-top: 12px;
  cursor: pointer;
  z-index: 2;
  position: absolute;
  right: 16px;
  top: 0;
}
.vc-chrome-toggle-icon-highlight {
  position: absolute;
  width: 24px;
  height: 28px;
  background: #eee;
  border-radius: 4px;
  top: 10px;
  left: 12px;
}
.vc-chrome-hue-wrap {
  position: relative;
  height: 18px;
  margin-bottom: 8px;
  border-radius: 15px;
  border: 1px solid rgba(33, 43, 54, 0.2);
}
.vc-chrome-alpha-wrap {
  position: relative;
  height: 18px;
  border-radius: 15px;
  border: 1px solid rgba(33, 43, 54, 0.2);
}
.vc-chrome-hue-wrap .vc-hue {
  border-radius: 15px;
}
.vc-chrome-alpha-wrap .vc-alpha-gradient {
  border-radius: 15px;
}
.vc-chrome-hue-wrap .vc-hue-picker, .vc-chrome-alpha-wrap .vc-alpha-picker {
  width: 14px;
  height: 14px;
  transform: translate(-6px, 0px);
  background-color: transparent;
  border: 3px solid #fff;
  border-radius: 50%;
  filter: drop-shadow(0px 1px 0px rgba(33, 43, 54, 0.05));
}
.vc-chrome-body {
  padding-top: 16px;
  background-color: #fff;
}
.vc-chrome-saturation-wrap {
  width: 100%;
  padding-bottom: 56.2%;
  position: relative;
  overflow: hidden;
  border-radius: 6px;
}
.vc-chrome-saturation-wrap .vc-saturation-circle {
  width: 18px;
  height: 18px;
  border: 3px solid #ffffff;
  box-shadow: none;
}

.vc-chrome-fields {
  gap: 4px;
}

.vc-chrome-fields .vc-input__input {
  font-size: 16px;
  color: #333;
  width: 100%;
  height: 46px;
  border: 1px solid #eee;
  border-radius: 6px;
  padding-left: 16px;
  text-align: center;
}
.vc-chrome-fields .vc-input__label {
  text-transform: uppercase;
  font-size: 12px;
  line-height: 12px;
  color: #999999;
  text-align: center;
  display: block;
  margin-top: 4px;
}

.vc-chrome__disable-alpha .vc-chrome-active-color {
  width: 18px;
  height: 18px;
}
.vc-chrome__disable-alpha .vc-chrome-color-wrap {
  width: 30px;
}
.vc-chrome__disable-alpha .vc-chrome-hue-wrap {
  margin-top: 4px;
  margin-bottom: 4px;
}
</style>
