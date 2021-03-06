<template>
  <div
    class="vue-ui-group"
    :class="{
      'has-indicator': indicator,
    }"
  >
    <div class="content">
      <slot/>
    </div>

    <div
      v-if="indicator && indicatorStyle"
      class="indicator"
      :style="{
        top: `${indicatorStyle.top}px`,
        left: `${indicatorStyle.left}px`,
        width: `${indicatorStyle.width}px`,
        height: `${indicatorStyle.height}px`,
      }"
    >
      <div class="content">
        <slot name="indicator"/>
      </div>
    </div>

    <resize-observer v-if="indicator" @notify="updateIndicator()"/>
  </div>
</template>

<script>
export default {
  name: 'VueGroup',

  provide () {
    return {
      VueGroup: {
        data: this.injection,
        setValue: this.setValue,
      },
    }
  },

  props: {
    value: {},

    indicator: {
      type: Boolean,
      default: false,
    },
  },

  data () {
    return {
      injection: {
        value: this.value,
      },

      indicatorStyle: null,
    }
  },

  watch: {
    value (value, oldValue) {
      if (value !== oldValue) {
        this.injection.value = value
        this.updateIndicator()
      }
    },
  },

  mounted () {
    this.updateIndicator()
  },

  methods: {
    setValue (value) {
      this.$emit('input', value)
    },

    updateIndicator () {
      this.$nextTick(() => {
        const el = this.$el.querySelector('.selected')
        if (el) {
          const offset = {
            top: el.offsetTop,
            left: el.offsetLeft,
            width: el.offsetWidth,
            height: el.offsetHeight,
          }
          let parent = el.offsetParent
          while (parent && parent !== this.$el) {
            offset.top += parent.offsetTop
            offset.left += parent.offsetLeft
            parent = parent.offsetParent
          }
          this.indicatorStyle = offset
        } else {
          this.indicatorStyle = null
        }
      })
    },
  },
}
</script>

<style lang="stylus">
@import "../style/imports"

indicator(direction)
  > .indicator
    padding-{direction} 1px
    > .content
      border-{direction} solid 2px rgba($vue-ui-color-dark, .7)
      .vue-ui-dark-mode &
        border-{direction}-color $vue-ui-color-light
  &.primary
    > .indicator
      > .content
        border-{direction}-color rgba($vue-ui-color-primary, .7)
  &.accent
    > .indicator
      > .content
        border-{direction}-color rgba($vue-ui-color-accent, .7)
        .vue-ui-dark-mode &
          border-{direction}-color lighten($vue-ui-color-accent, 60%)

.vue-ui-group
  position relative

  > .content
    h-box()
    align-items stretch
    justify-content center

  &.start
    > .content
      justify-content flex-start

  &.end
    > .content
      justify-content flex-end

  &.vertical
    > .content
      flex-direction column

  &.inline
    display inline-block
    vertical-align middle

  > .indicator
    $time = .15s
    position absolute
    transition top $time ease-in-out, left $time ease-in-out, width $time ease-in-out, height $time ease-in-out
    pointer-events none
    box-sizing border-box
    h-box()
    box-center()

    > .content
      width 100%
      height 100%

  &:not(.vertical)
    &:not(.top-indicator)
      indicator(bottom)
    &.top-indicator
      indicator(top)
    &.small-indicator
      > .indicator
        > .content
          width 12px

  &.vertical
    &:not(.left-indicator)
      indicator(right)
    &.left-indicator
      indicator(left)
    &.small-indicator
      > .indicator
        > .content
          height 12px

  &.extend
    > .content
      > .vue-ui-button
        &:not(.icon-button)
          flex 100% 1 1
          width 0

</style>
