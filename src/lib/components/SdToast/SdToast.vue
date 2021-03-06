<template>
  <teleport to="#app">
    <transition name="toast">
      <div ref="currentToast" :class="['sd--toast', classes, 'sd--elevation--12']" v-if="active">
        <button class="sd--toast__close" @click="() => handleClose()" v-if="dismissable"><sd-icon name="close"/></button>
        <div class="sd--toast__content">
          <slot/>
        </div>
      </div>
    </transition>
  </teleport>
</template>

<script>
import { computed, watch, reactive } from 'vue'

export default {
  name: 'SdToast',
  props: {
    active: Boolean,
    theme: {
      type: String,
      default: 'dark'
    },
    placement: {
      type: String,
      default: 'bottom'
    },
    duration: {
      type: Number,
      default: 3000
    },
    dismissable: Boolean,
    persistant: Boolean
  },
  emits: ['opened', 'closed', 'update:active'],

  setup (props, { emit }) {
    const state = reactive({
      active: false
    })

    let currentToast = null
    let timeout = null

    const classes = computed(() => {
      const placementModifier = `sd--toast--${props.placement}`
      const themeModifier = `sd--toast__${props.theme}`
      return {
        [placementModifier]: true,
        [themeModifier]: true,
        'is--active': props.active
      }
    })

    const handleClose = () => {
      emit('update:active', false)
    }

    const promiseToast = (duration, persist) => {
      return new Promise(resolve => {
        currentToast = {
          destroy: () => {
            currentToast = null
            resolve()
          }
        }
        if (duration !== Infinity) {
          timeout = window.setTimeout(() => {
            destroyToast()
            if (!persist) {
              emit('update:active', false)
            }
          }, duration)
        }
      })
    }

    const destroyToast = () => {
      return new Promise(resolve => {
        if (currentToast.value) {
          window.clearTimeout(timeout)
          window.setTimeout(resolve, 400)
          emit('update:active', false)
        } else {
          resolve()
        }
      })
    }

    const makeToast = (duration, persist) => {
      if (currentToast) {
        return destroyToast().then(() => {
          return promiseToast(duration, persist)
        })
      }

      return promiseToast(duration, persist)
    }

    watch(() => props.active, () => {
      if (props.active) {
        makeToast(props.duration, props.persistant).then(() => {
          emit('opened')
          emit('update:active', false)
        })
      } else {
        destroyToast()
        emit('closed')
      }
    })

    return { classes, currentToast, state, destroyToast, handleClose }
  }
}
</script>

<style lang="scss">
  @import '../../scss/variables';
  @import '../../scss/mixins';
  @import '../../scss/breakpoints';
  @import '../SdElevation/elevation.scss';
  .sd--toast{
    position: fixed;
    z-index: 9999;
    transition: transform .2s ease-in-out, opacity .2s ease-in-out;
    display:flex;
    border-radius: 3px;
    &__close{
      display: flex;
      align-items: center;
      padding:8px;
      vertical-align: middle;
      outline:none;
      background:none;
      border:none;
    }

    &__content{
      z-index: 9999;
      min-width: 300px;
      display:flex;
      align-items: center;
      font-size: 14px;
      padding:4px 4px 4px 16px;
      flex-grow: 2;
      border-radius: 3px;
    }

  @each $state, $color in $sd-color-global {
    &__#{$state} {
      .sd--toast__content{
        background-color: var(--#{$state});
        color: var(--#{$state}-text);
      }
      .sd--toast__close{
        background-color: var(--#{$state}-accent);
        color: var(--#{$state}-text);
        border-radius: 3px 0 0 3px;
        margin-right: -3px;
      }
    }
  }

  &--top {
    top: 16px;
    left: 50%;
    transform: translateX(-50%) translateY(0);
  }

  &--bottom {
    bottom: 16px;
    left: 50%;
    transform: translateX(-50%) translateY(0);
  }

  &--top-left {
    top: 16px;
    left: 16px;
    transform: translateY(0);
  }

  &--top-right {
    right: 16px;
    top:16px;
    transform: translateY(0);
  }

  &--bottom-left {
    left: 16px;
    bottom: 16px;
    transform: translateY(0);
  }

  &--bottom-right {
    right: 16px;
    bottom: 16px;
    transform: translateY(0);
  }

    @include breakpoint-down('sm') {
    &--top-right, &--top-left, &--top {
      left:16px;
      right:16px;
      top: 16px;
      transform: translateX(0) translateY(0);
    }
    &--bottom-right, &--bottom-left, &--bottom {
      left:16px;
      right:16px;
      bottom: 16px;
      transform: translateX(0) translateY(0);
    }
  }
}

.toast-enter-active, .toast-leave-active{
  transition: transform .28s ease-in-out, opacity .2s ease-in-out;
}

.toast-leave-to, .toast-enter-from{
  &.sd--toast{
    opacity: 0;
    &--top {
      transform: translateX(-50%) translateY(-100px);

      @include breakpoint-down('sm') {
        transform: translateX(0) translateY(-100px);
      }
    }
    &--bottom {
      transform: translateX(-50%) translateY(100px);

      @include breakpoint-down('sm') {
        transform: translateX(0) translateY(100px);
      }
    }
    &--top-left, &--top-right {
      transform: translateY(-100px);
    }
    &--bottom-left, &--bottom-right {
      transform: translateY(100px);
    }
  }
}
</style>
