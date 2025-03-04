<!--
 * Copyright (c) 2022 - present TinyVue Authors.
 * Copyright (c) 2022 - present Huawei Cloud Computing Technologies Co., Ltd.
 *
 * Use of this source code is governed by an MIT-style license.
 *
 * THE OPEN SOURCE SOFTWARE IN THIS PRODUCT IS DISTRIBUTED IN THE HOPE THAT IT WILL BE USEFUL,
 * BUT WITHOUT ANY WARRANTY, WITHOUT EVEN THE IMPLIED WARRANTY OF MERCHANTABILITY OR FITNESS FOR
 * A PARTICULAR PURPOSE. SEE THE APPLICABLE LICENSES FOR MORE DETAILS.
 *
 -->
<script lang="tsx">
import { $prefix, setup, h, defineComponent } from '@opentiny/vue-common'
import { t } from '@opentiny/vue-locale'
import { renderless, api } from '@opentiny/vue-renderless/tab-nav/vue'
import Dropdown from '@opentiny/vue-dropdown'
import DropdownMenu from '@opentiny/vue-dropdown-menu'
import DropdownItem from '@opentiny/vue-dropdown-item'
import Tooltip from '@opentiny/vue-tooltip'
import { iconChevronLeft, iconChevronRight, iconClose } from '@opentiny/vue-icon'
import type { ITabNavApi } from '@opentiny/vue-renderless/types/tab-nav.type'
import TabBar from './tab-bar.vue'
import { tabNavPcProps } from './index'

export default defineComponent({
  name: $prefix + 'TabNav',
  components: {
    TabBar,
    Dropdown,
    DropdownMenu,
    DropdownItem,
    Tooltip,
    IconChevronLeft: iconChevronLeft(),
    IconChevronRight: iconChevronRight(),
    IconClose: iconClose()
  },
  props: tabNavPcProps,
  inject: ['moreIcon'],
  setup(props, context) {
    return setup({ props, context, renderless, api, mono: true }) as unknown as ITabNavApi
  },
  render() {
    const {
      state,
      tabStyle,
      editable,
      stretch,
      showPanesCount,
      onTabClick,
      onTabRemove,
      scrollNext,
      scrollPrev,
      changeTab,
      setFocus,
      removeFocus,
      showMoreTabs,
      popperClass,
      overflowTitle,
      titleWidth,
      handleTitleMouseenter,
      handleTitleMouseleave,
      // tiny 新增
      moreIcon,
      tooltipConfig,
      panelMaxHeight,
      panelWidth
    } = this
    let { panes } = this

    const spans = [
      <span class={['tiny-tabs__nav-prev', state.scrollable.prev ? '' : 'is-disabled']} onClick={scrollPrev}>
        <icon-chevron-left />
      </span>,
      !showMoreTabs ? (
        <span class={['tiny-tabs__nav-next', state.scrollable.next ? '' : 'is-disabled']} onClick={scrollNext}>
          <icon-chevron-right />
        </span>
      ) : null
    ]

    const scrollBtn = state.scrollable ? spans : null
    let moreTabs = null

    if (showMoreTabs && state.scrollable) {
      const reference = () =>
        moreIcon ? (
          <span class="tiny-tabs__more-icon">{moreIcon() || t('ui.tabs.moreItem')}</span>
        ) : (
          <span class="tiny-tabs__more">{t('ui.tabs.moreItem')}</span>
        )

      const dropdowpList = panes.slice(showPanesCount)
      const isShowDropDown = showPanesCount !== -1 && Boolean(dropdowpList.length)

      const menuSlot = () =>
        isShowDropDown
          ? dropdowpList.map((pane, index) => {
              const tabName = pane.name || pane.state.index || index
              const tabLabelContent = pane.$slots.title || pane.title
              const tabindex = pane.state.active ? 0 : -1

              pane.state.index = `${index + showPanesCount}`

              return (
                <dropdown-item
                  class="tiny-tabs__more-item"
                  tabindex={tabindex}
                  onFocus={() => {
                    setFocus()
                  }}
                  onBlur={() => {
                    removeFocus()
                  }}
                  onClick={(e) => {
                    removeFocus()
                    onTabClick(pane, tabName, e)
                  }}>
                  {typeof tabLabelContent === 'function' ? tabLabelContent() : tabLabelContent}
                </dropdown-item>
              )
            })
          : null

      const dropdownSlot = () =>
        isShowDropDown
          ? h(DropdownMenu, {
              attrs: {
                popperClass: 'tiny-tabs-dropdown tiny-tabs__more-dropdown' + (popperClass ? ' ' + popperClass : ''),
                placement: 'bottom-start',
                style: { maxHeight: panelMaxHeight, width: panelWidth }
              },
              scopedSlots: { default: menuSlot }
            })
          : null

      // tiny 新增：防止没有内容也显示下拉框。  aui 此处为Popover组件实现
      moreTabs = (
        <div class="tiny-tabs__more-container" ref="more">
          {isShowDropDown
            ? h(Dropdown, {
                attrs: {
                  trigger: 'hover'
                },
                scopedSlots: { default: reference, dropdown: dropdownSlot }
              })
            : reference()}
        </div>
      )
    }

    const tabs = panes.map((pane, index) => {
      let tabName = pane.name || pane.state.index || index
      const withClose = pane.state.isClosable || editable

      pane.state.index = `${index}`

      const btnClose = withClose ? (
        <span class="tiny-tabs__icon-close">
          <icon-close
            onClick={(e) => {
              onTabRemove(pane, e)
            }}
          />
        </span>
      ) : null

      const getTabTitle = (title) => {
        return h(
          'span',
          {
            class: { 'tiny-tabs__item-title': true },
            style: {
              'max-width': titleWidth
            },
            on: {
              mouseenter(e) {
                handleTitleMouseenter(e, title)
              },
              mouseleave(e) {
                handleTitleMouseleave(e)
              }
            }
          },
          [title]
        )
      }

      const tipComp = () =>
        tooltipConfig === 'title'
          ? h('span', { class: 'tiny-tabs__item__title', attrs: { title: pane.title } }, [pane.title])
          : h(
              Tooltip,
              {
                class: 'tiny-tabs__item__title',
                props: {
                  content: pane.title,
                  ...tooltipConfig
                }
              },
              [h('span', {}, [pane.title])]
            )
      const toolTipComp = () =>
        tooltipConfig ? tipComp() : h('span', { class: 'tiny-tabs__item__title' }, [pane.title])
      const itemsSeparator = <span class="tiny-tabs__item-separator"></span>

      const tabLabelContent = pane.$slots.title ? pane.$slots.title() : toolTipComp()
      const tabindex = pane.state.active ? 0 : -1

      return h(
        'div',
        {
          ref: `tabs-${tabName}`,
          key: `tab-${tabName}-${index}`,
          class: {
            'tiny-tabs__item': true,
            'tiny-tabs__item-separator-space': state.separator, // 注：tiny-tabs__item-base类名不需要同步aui，会改变tabs的基本样式
            [`is-${state.rootTabs.position}`]: true,
            'is-active': pane.state.active,
            'is-disabled': pane.disabled,
            'is-closable': withClose,
            'is-focus': state.isFocus
          },
          attrs: {
            id: `tab-${tabName}`,
            'aria-controls': `pane-${tabName}`,
            'data-index': index + 1,
            role: 'tab',
            'aria-selected': pane.state.active,
            tabindex,
            refInFor: true
          },
          on: {
            focus() {
              setFocus()
            },
            blur() {
              removeFocus()
            },
            click(e) {
              removeFocus()
              onTabClick(pane, tabName, e)
            },
            keydown(e) {
              if (withClose && (e.keyCode === 46 || e.keyCode === 8)) {
                onTabRemove(pane, e)
              }
            }
          }
        },
        [overflowTitle ? getTabTitle(tabLabelContent) : tabLabelContent, btnClose, state.separator && itemsSeparator]
      )
    })

    // 根据生成的子元素宽度动态设置tabslist盒子的padding-right
    const paddingRight = this.$refs.more ? `${this.$refs.more.offsetWidth}px` : '46px'

    return (
      <div
        style={showMoreTabs ? { paddingRight } : {}}
        class={[
          'tiny-tabs__nav-wrap',
          !state.separator && 'tiny-tabs__nav-wrap-not-separator',
          state.scrollable ? 'is-scrollable' : '',
          showMoreTabs ? 'is-show-more' : '',
          `is-${state.rootTabs.position}`
        ]}>
        {[scrollBtn, moreTabs]}
        <div class={['tiny-tabs__nav-scroll']} ref="navScroll">
          <div
            class={[
              'tiny-tabs__nav',
              `is-${state.rootTabs.position}`,
              stretch && ['top', 'bottom'].includes(state.rootTabs.position) ? 'is-stretch' : '',
              !tabStyle ? 'is-show-active-bar' : ''
            ]}
            ref="nav"
            style={state.navStyle}
            role="tablist"
            on-keydown={changeTab}>
            {!tabStyle ? <tab-bar ref="tabBar" tabs={panes} /> : null}
            {tabs}
          </div>
        </div>

        {overflowTitle ? (
          <Tooltip
            ref="tooltip"
            v-model={state.tooltipVisible}
            manual={true}
            effect="light"
            content={state.tooltipContent}
            placement="top"></Tooltip>
        ) : null}
      </div>
    )
  }
})
</script>
