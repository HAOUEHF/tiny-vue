<template>
  <div data-tag="tiny-drawer" class="text-sm" v-show="visible">
    <!-- mask -->
    <div
      ref="mask"
      v-if="mask"
      :class="
        m('fixed z-50 left-0 right-0 top-0 bottom-0 w-full h-full transition-opacity ease-linear duration-200', {
          'bg-color-bg-7': state.toggle
        })
      "
      @click="handleClose('mask')"
    ></div>
    <!-- main -->
    <div
      data-tag="tiny-drawer-main"
      ref="drawerBox"
      :class="
        m(
          'fixed w-full max-w-full flex flex-col bg-color-bg-1 z-50 shadow-sm border-color-border-separator',
          { 'transition-all ease-linear duration-200': !state.dragEvent.isDrag },
          { 'h-full': ['left', 'right'].includes(placement) },
          { 'max-h-full': ['top', 'bottom'].includes(placement) },
          { 'left-0 bottom-0 translate-y-full border-t-0.5 rounded-t-lg': placement === 'bottom' },
          { 'left-0 top-0 -translate-y-full border-b-0.5 rounded-b-lg': placement === 'top' },
          { 'translate-y-0': ['top', 'bottom'].includes(placement) && state.toggle },
          { 'left-0 top-0 -translate-x-full border-r-0.5 rounded-r-lg': placement === 'left' },
          { 'right-0 top-0 translate-x-full border-l-0.5 rounded-l-lg': placement === 'right' },
          { 'translate-x-0': ['left', 'right'].includes(placement) && state.toggle },
          customClass
        )
      "
      :style="{
        width: ['left', 'right'].includes(placement) ? state.computedWidth : null,
        height: ['top', 'bottom'].includes(placement) && dragable && state.height ? state.height + 'px' : null
      }"
    >
      <div
        data-tag="drawer-drag-bar"
        ref="dragBar"
        v-if="dragable"
        :class="[
          'absolute',
          ['left', 'right'].includes(placement) && 'cursor-e-resize h-full top-0 w-2',
          ['top', 'bottom'].includes(placement) && 'cursor-n-resize w-full h-2 left-0',
          placement === 'left' && '-right-1',
          placement === 'right' && '-left-1',
          placement === 'top' && '-bottom-1',
          placement === 'bottom' && '-top-1'
        ]"
      ></div>
      <div :class="['flex-auto flex-col flex max-h-full overflow-hidden']">
        <!-- header -->
        <div
          data-tag="drawer-header"
          ref="header"
          v-if="showHeader"
          class="flex-none flex leading-6 p-4 text-base items-center"
        >
          <slot name="header">
            <div v-if="title" class="max-w-[80%] pr-4 text-left truncate text-color-text-primary">{{ title }}</div>
            <div class="flex-1 flex items-center justify-end">
              <slot name="header-right">
                <IconClose
                  custom-class="h-5 w-5 cursor-pointer"
                  class="fill-color-icon-primary"
                  @click="handleClose('close')"
                ></IconClose>
              </slot>
            </div>
          </slot>
        </div>
        <!-- body -->
        <div data-tag="drawer-body" ref="body" :class="['flex-auto overflow-auto', { 'flex flex-col': flex }]">
          <slot></slot>
        </div>
        <!-- footer -->
        <div data-tag="drawer-footer" ref="footer" v-if="showFooter" class="px-4 py-3">
          <div class="flex-1 text-right">
            <slot name="footer">
              <tiny-button tiny_mode="mobile-first" @click="handleClose('cancel')">{{
                t('ui.button.cancel')
              }}</tiny-button>
              <tiny-button tiny_mode="mobile-first" class="ml-2" type="primary" @click="handleClose('confirm')">{{
                t('ui.button.confirm')
              }}</tiny-button>
            </slot>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import { renderless, api } from '@opentiny/vue-renderless/drawer/vue'
import { setup, props, defineComponent } from '@opentiny/vue-common'
import { IconClose } from '@opentiny/vue-icon'
import Button from '@opentiny/vue-button'

export default defineComponent({
  components: {
    TinyButton: Button,
    IconClose: IconClose()
  },
  props: [
    ...props,
    'visible',
    'title',
    'showHeader',
    'showFooter',
    'customClass',
    'placement',
    'width',
    'mask',
    'dragable',
    'maskClosable',
    'lockScroll',
    'flex',
    'beforeClose'
  ],
  setup(props, context) {
    return setup({ props, context, renderless, api })
  }
})
</script>
