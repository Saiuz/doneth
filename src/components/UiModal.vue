<template>
<transition :name="toggleTransition" @after-enter="onEnter" @after-leave="onLeave">
  <div class="ui-modal ui-modal__mask" :class="classes" :role="role" v-show="isOpen">
    <div class="ui-modal__wrapper" ref="backdrop" :class="{ 'has-dummy-scrollbar': preventShift }" @click.self="closeModal($event)">
      <div class="ui-modal__container" ref="container" tabindex="-1" @keydown.esc="dismissOnEsc && closeModal($event)">
        <div class="ui-modal__header" v-if="!removeHeader">
          <slot name="header">
            <h1 class="ui-modal__header-text">{{ title }}</h1>
          </slot>

          <div class="ui-modal__close-button">
            <button @click="closeModal" v-if="dismissOnCloseButton && !removeCloseButton">&times;</button>
          </div>
        </div>

        <div class="ui-modal__body">
          <slot></slot>
        </div>

        <div class="ui-modal__footer" v-if="hasFooter">
          <slot name="footer"></slot>
        </div>

        <div class="ui-modal__focus-redirect" tabindex="0" @focus.stop="redirectFocus"></div>
      </div>
    </div>
  </div>
</transition>
</template>

<script>
/**
 * UiModal - Keen UI
 * Credit:
 * https://github.com/JosephusPaye/Keen-UI/blob/master/src/UiModal.vue
 */
import classlist from './helpers/classlist'
import {mapState, mapMutations} from 'vuex'
export default {
  name: 'ui-modal',

  props: {
    modalId: {
      type: String,
      default: null
    },
    title: {
      type: String,
      default: 'UiModal title'
    },
    size: {
      type: String,
      default: 'normal' // 'small', 'normal', or 'large'
    },
    role: {
      type: String,
      default: 'dialog' // 'dialog' or 'alertdialog'
    },
    transition: {
      type: String,
      default: 'scale' // 'scale', or 'fade'
    },
    removeHeader: {
      type: Boolean,
      default: false
    },
    removeCloseButton: {
      type: Boolean,
      default: false
    },
    preventShift: {
      type: Boolean,
      default: false
    },
    dismissible: {
      type: Boolean,
      default: true
    },
    dismissOn: {
      type: String,
      default: 'backdrop esc close-button'
    }
  },

  data () {
    return {
      lastfocusedElement: null
    }
  },

  computed: {
    ...mapState(['modalOpen']),
    isOpen () {
      return this.modalOpen === this.modalId
    },
    classes () {
      return [
        `ui-modal--size-${this.size}`,
        {
          'has-footer': this.hasFooter
        },
        {
          'is-open': this.modalOpen
        }
      ]
    },

    hasFooter () {
      return Boolean(this.$slots.footer)
    },

    toggleTransition () {
      return `ui-modal--transition-${this.transition}`
    },

    dismissOnBackdrop () {
      return this.dismissOn.indexOf('backdrop') > -1
    },

    dismissOnCloseButton () {
      return this.dismissOn.indexOf('close-button') > -1
    },

    dismissOnEsc () {
      return this.dismissOn.indexOf('esc') > -1
    }
  },

  watch: {
    modalOpen () {
      this.$nextTick(() => {
        this[this.modalOpen ? 'onOpen' : 'onClose']()
      })
    }
  },

  beforeDestroy () {
    if (this.modalOpen) {
      this.teardownModal()
    }
  },

  methods: {
    ...mapMutations({setModal: 'SET_MODAL', setEditMember: 'SET_EDIT_MEMBER'}),
    closeModal (e) {
      this.setModal(false)
      this.setEditMember(null)
    },

    onOpen () {
      this.lastfocusedElement = document.activeElement
      this.$refs.container.focus()

      classlist.add(document.body, 'ui-modal--is-open')
      document.addEventListener('focus', this.restrictFocus, true)

      this.$emit('open')
    },

    onClose () {
      this.teardownModal()
      this.$emit('close')
    },

    redirectFocus () {
      this.$refs.container.focus()
    },

    restrictFocus (e) {
      if (!this.$refs.container.contains(e.target)) {
        e.stopPropagation()
        this.$refs.container.focus()
      }
    },

    teardownModal () {
      // classlist.remove(document.body, 'ui-modal--is-open')
      document.removeEventListener('focus', this.restrictFocus, true)

      if (this.lastfocusedElement) {
        this.lastfocusedElement.focus()
      }
    },

    onEnter () {
      this.$emit('reveal')
    },

    onLeave () {
      this.$emit('hide')

      classlist.remove(document.body, 'ui-modal--is-open')
    }
  }
}
</script>

<style lang="scss">
@import '../scss/variables';

$ui-modal-transition-duration: 0.3s !default;
$ui-modal-mask-background: rgba(black, 0.5) !default;
$ui-modal-header-height: 56px;
$ui-modal-footer-height: 70px;

$ui-modal-font-size: 14px;
$ui-modal-header-font-size: 18px;
$ui-default-border-radius: $border-radius;
$z-index-modal: 9999;

.ui-modal {
    font-family: inherit;// $font-stack;
    font-size: $ui-modal-font-size;

    &.has-footer {
        .ui-modal__body {
            max-height: calc(100vh - #{$ui-modal-header-height + $ui-modal-footer-height});
        }
    }
    &:not(.has-footer) {
        .ui-modal__body {
            padding: 16px 24px 24px;
        }
    }
}

.ui-modal--is-open {
    overflow: hidden;
}

.ui-modal__mask {
    background-color: $ui-modal-mask-background;
    display: table;
    height: 100%;
    left: 0;
    position: fixed;
    top: 0;
    transition: opacity $ui-modal-transition-duration ease;
    width: 100%;
    z-index: $z-index-modal;
}

.ui-modal__wrapper {
    display: table-cell;
    vertical-align: middle;
    overflow-x: hidden;

    &.has-dummy-scrollbar {
        overflow-y: scroll;
    }
}

.ui-modal__container {
    background-color: white;
    border-radius: $ui-default-border-radius;
    box-shadow: 0 2px 8px rgba(black, 0.33);
    margin: 0 auto;
    max-height: 100vh;
    max-width: 100vw;
    outline: none;
    overflow: hidden;
    padding: 0;
    transition: all $ui-modal-transition-duration ease;
    width: 528px;
}

.ui-modal__header {
    align-items: center;
    background-color: #F5F5F5;
    display: flex;
    height: $ui-modal-header-height;
    padding: 0 24px;
    position: relative;
    z-index: 1;
}

.ui-modal__header-text {
    align-items: center;
    display: flex;
    flex-grow: 1;
    font-size: $ui-modal-header-font-size;
    font-weight: normal;
    margin: 0;
}

.ui-modal__close-button {
    margin-left: auto;
    margin-right: -8px;

    button {
      background: transparent;
      border: 0;
      cursor: pointer;
      font-size: 30pt;
      line-height: 23pt;
      margin: 0;
      outline: 0;
      padding: 3px 0 10px 10px;
    }
}

.ui-modal__body {
    max-height: calc(100vh - #{$ui-modal-header-height});
    overflow-y: auto;
    padding: 16px 24px;
}

.ui-modal__footer {
    align-items: center;
    background-color: #F5F5F5;
    display: flex;
    height: $ui-modal-footer-height;
    justify-content: flex-end;
    padding: 0 24px;

    .btn {
        margin-left: 8px;
        min-width: 100px;

        &:first-child {
            margin-left: 0;
        }
    }
}

// ================================================
// Sizes
// ================================================

.ui-modal--size-large {
    .ui-modal__container {
        width: 848px;
    }
}

.ui-modal--size-small {
    .ui-modal__container {
        width: 320px;
    }
}

// ================================================
// Transitions
// ================================================

.ui-modal--transition-fade-enter,
.ui-modal--transition-fade-leave-active {
    opacity: 0;
}

.ui-modal--transition-scale-enter,
.ui-modal--transition-scale-leave-active {
    opacity: 0;

    .ui-modal__container {
        transform: scale(1.1);
    }
}
</style>
