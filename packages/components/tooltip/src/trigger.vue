<template>
  <el-popper-trigger
    :id="id"
    :virtual-ref="virtualRef"
    :open="open"
    :virtual-triggering="virtualTriggering"
    :class="ns.e('trigger')"
    @blur="onBlur"
    @click="onClick"
    @contextmenu="onContextMenu"
    @focus="onFocus"
    @mouseenter="onMouseenter"
    @mouseleave="onMouseleave"
    @keydown="onKeydown"
  >
    <slot />
  </el-popper-trigger>
</template>

<script lang="ts">
import { defineComponent, inject, ref, toRef, unref } from 'vue'
import { ElPopperTrigger } from '@element-plus/components/popper'
import { composeEventHandlers } from '@element-plus/utils'
import { useNamespace } from '@element-plus/hooks'
import { EVENT_CODE } from '@element-plus/constants'
import { TOOLTIP_INJECTION_KEY } from './tokens'
import { useTooltipTriggerProps } from './tooltip'
import { whenTrigger } from './utils'

import type { OnlyChildExpose } from '@element-plus/components/slot'

export default defineComponent({
  name: 'ElTooltipTrigger',
  components: {
    ElPopperTrigger,
  },
  props: useTooltipTriggerProps,
  setup(props) {
    const ns = useNamespace('tooltip')
    const { controlled, id, open, onOpen, onClose, onToggle } = inject(
      TOOLTIP_INJECTION_KEY,
      undefined
    )!
    const triggerRef = ref<OnlyChildExpose | null>(null)

    const stopWhenControlledOrDisabled = () => {
      if (unref(controlled) || props.disabled) {
        return true
      }
    }
    const trigger = toRef(props, 'trigger')
    const onMouseenter = composeEventHandlers(
      stopWhenControlledOrDisabled,
      whenTrigger(trigger, 'hover', onOpen)
    )
    const onMouseleave = composeEventHandlers(
      stopWhenControlledOrDisabled,
      whenTrigger(trigger, 'hover', onClose)
    )
    const onClick = composeEventHandlers(
      stopWhenControlledOrDisabled,
      whenTrigger(trigger, 'click', (e) => {
        // distinguish left click
        if ((e as MouseEvent).button === 0) {
          // 有时候鼠标左键单击下拉面板会被关掉，可能原作者想法是右键关闭但是写反了
          // onToggle(e)
        }
      })
    )

    const onFocus = composeEventHandlers(
      stopWhenControlledOrDisabled,
      whenTrigger(trigger, 'focus', onOpen)
    )

    const onBlur = composeEventHandlers(
      stopWhenControlledOrDisabled,
      whenTrigger(trigger, 'focus', onClose)
    )

    const onContextMenu = composeEventHandlers(
      stopWhenControlledOrDisabled,
      whenTrigger(trigger, 'contextmenu', (e: Event) => {
        e.preventDefault()
        onToggle(e)
      })
    )

    const onKeydown = composeEventHandlers(
      stopWhenControlledOrDisabled,
      (e: KeyboardEvent) => {
        const { code } = e
        if (props.triggerKeys.includes(code)) {
          // 作者原想法是空格选中，但是与我们的打字方式空格确定冲突了
          if (code == EVENT_CODE.space) return

          e.preventDefault()
          onToggle(e)
        }
      }
    )

    return {
      onBlur,
      onContextMenu,
      onFocus,
      onMouseenter,
      onMouseleave,
      onClick,
      onKeydown,
      open,
      id,
      triggerRef,
      ns,
    }
  },
})
</script>
