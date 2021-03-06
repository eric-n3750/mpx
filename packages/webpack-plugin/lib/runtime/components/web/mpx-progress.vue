<script>
  import getInnerListeners, { getCustomEvent } from '@mpxjs/webpack-plugin/lib/runtime/components/web/getInnerListeners'


  export default {
    name: 'mpx-progress',
    props: {
      percent: {
        type: Number,
        default: 0
      },
      showInfo: {
        type: Boolean,
        default: false
      },
      borderRadius: {
        type: [Number, String],
        default: 0
      },
      fontSize: {
        type: [Number, String],
        default: 16
      },
      strokeWidth: {
        type: [Number, String],
        default: 6
      },
      color: {
        type: String,
        default: '#09BB07'
      },
      activeColor: {
        type: String,
        default: '#09BB07'
      },
      backgroundColor: {
        type: String,
        default: '#EBEBEB'
      },
      active: {
        type: Boolean,
        default: false
      },
      activeMode: {
        type: String,
        default: 'backwards'
      },
      duration: {
        type: Number,
        default: 30
      }
    },
    data () {
      return {
        innerPercent: 0,
        transition: 'none'
      }
    },
    mounted () {
      this.$watch(() => {
        return this.percent
      }, () => {
        if (this.active) {
          this.doTransition()
        } else {
          this.innerPercent = this.percent
        }
      }, {
        immediate: true
      })
    },
    methods: {
      doTransition () {
        const from = this.activeMode === 'backwards' ? 0 : this.innerPercent
        const to = this.percent
        const duration = this.duration * Math.abs(to - from)
        this.innerPercent = from
        this.transition = 'none'
        setTimeout(() => {
          this.innerPercent = to
          this.transition = `all ${duration}ms`
        })
      },
      resetTransition () {
        this.transition = 'none'
      }
    },
    render (createElement) {
      const children = []
      const strokeWidthStr = isNaN(+this.strokeWidth) ? this.strokeWidth : this.strokeWidth + 'px'
      const fontSizeStr = isNaN(+this.fontSize) ? this.fontSize : this.fontSize + 'px'
      const borderRadiusStr = isNaN(+this.borderRadius) ? this.borderRadius : this.borderRadius + 'px'

      const progress = createElement('div', {
        class: 'progress',
        style: {
          height: strokeWidthStr,
          backgroundColor: this.activeColor || this.color,
          transform: `scaleX(${this.innerPercent / 100})`,
          transition: this.transition
        },
        ref: 'progress',
        on: {
          transitionend: () => {
            this.$emit('activeend', getCustomEvent('activeend'))
          }
        }
      })
      const progressContainer = createElement('div', {
        class: 'container',
        style: {
          height: strokeWidthStr,
          borderRadius: borderRadiusStr,
          backgroundColor: this.backgroundColor
        }
      }, [progress])

      children.push(progressContainer)

      if (this.showInfo) {
        const info = createElement('div', {
          class: 'info',
          style: {
            fontSize: fontSizeStr
          }
        }, this.innerPercent + '%')
        children.push(info)
      }

      children.push(...(this.$slots.default || []))

      const data = {
        class: ['mpx-progress'],
        on: getInnerListeners(this)
      }
      return createElement('div', data, children)
    }
  }
</script>

<style lang="stylus">
  .mpx-progress
    display flex
    align-items center
    margin 10px

    .container
      flex: 1
      position relative
      overflow hidden

    .progress
      position absolute
      top 0
      left 0
      width 100%
      transform-origin 0 0

    .info
      padding-left 20px
</style>
