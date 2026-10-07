<template>
  <app-dialog
    v-model="open"
    :title="dialog.title"
    max-width="450"
    :no-actions="dialog.footerButtons.length === 0"
  >
    <v-card-text>
      <div
        v-for="(item, index) in dialog.items"
        :key="`item-${index}`"
        class="mb-2"
      >
        <v-row v-if="item.type === 'text'">
          <v-col>
            {{ item.text }}
          </v-col>
        </v-row>
        <v-row v-else-if="item.type === 'button'">
          <v-col>
            <v-btn
              :color="!isHexColor(item.color) ? item.color : undefined"
              :style="computedStyle(item.color)"
              block
              @click="handleClick(item)"
            >
              {{ item.text }}
            </v-btn>
          </v-col>
        </v-row>
        <v-row
          v-else-if="item.type === 'button_group'"
          class="mx-n2"
        >
          <v-col
            v-for="(btn, btnIndex) in item.buttons"
            :key="`btn-${index}-${btnIndex}`"
            class="px-2"
          >
            <v-btn
              :color="!isHexColor(btn.color) ? btn.color : undefined"
              :style="computedStyle(btn.color)"
              block
              @click="handleClick(btn)"
            >
              {{ btn.text }}
            </v-btn>
          </v-col>
        </v-row>
      </div>
    </v-card-text>
    <template #actions>
      <v-spacer />
      <app-btn
        v-for="(button, index) in dialog.footerButtons"
        :key="`button-${index}`"
        :color="button.color ?? 'primary'"
        type="button"
        @click="handleClick(button)"
      >
        {{ button.text }}
      </app-btn>
    </template>
  </app-dialog>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import StateMixin from '@/mixins/state'
import type { PromptDialog, PromptDialogButton } from '@/store/console/types'

@Component({})
export default class ActionCommandPromptDialog extends Mixins(StateMixin) {
  get dialog (): PromptDialog {
    return this.$typedState.console.promptDialog
  }

  get open (): boolean {
    return this.dialog.open
  }

  set open (value: boolean) {
    if (!value) {
      this.sendGcode('RESPOND TYPE=command MSG="action:prompt_end"')
    }
  }

  isHexColor (color: string | undefined): boolean {
    return typeof color === 'string' && /^#(?:[0-9a-f]{3}){1,2}$/i.test(color)
  }

  computedStyle (color: string | undefined) {
    if (this.isHexColor(color)) {
      const bgColor = color
      const textColor = this.computeTextColor(bgColor)
      return {
        backgroundColor: bgColor,
        color: textColor
      }
    }
    return {}
  }

  computeTextColor (backgroundColor: string | undefined): string {
    if (!backgroundColor) {
      return 'white'
    }
    const hex = backgroundColor.startsWith('#') ? backgroundColor : '#' + backgroundColor

    const shorthandRegex = /^#?([a-f\d])([a-f\d])([a-f\d])$/i
    const fullHex = hex.replace(shorthandRegex, (m, r, g, b) => r + r + g + g + b + b)

    const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(fullHex)
    if (!result) return 'white'

    const r = parseInt(result[1], 16)
    const g = parseInt(result[2], 16)
    const b = parseInt(result[3], 16)

    const gray = r * 0.299 + g * 0.587 + b * 0.114

    return gray > 186 ? 'black' : 'white'
  }

  handleClick (button: PromptDialogButton) {
    this.sendGcode(button.command || button.text)
  }
}
</script>
