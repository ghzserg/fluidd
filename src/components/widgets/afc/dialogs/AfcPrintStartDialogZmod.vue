<template>
  <div v-if="zmod">
    <v-divider />
    <app-setting
      :title="$t('app.afc.PrintStartDialog.Leveling')"
      :r-cols="2"
    >
      <v-switch
        :input-value="leveling"
        hide-details
        @change="save('print_leveling', $event)"
        @click.native.stop
      />
    </app-setting>
    <template v-if="c5pro">
      <v-divider />
      <app-setting
        :title="$t('app.afc.PrintStartDialog.AutoPa')"
        :r-cols="2"
      >
        <v-switch
          :input-value="autoPa"
          hide-details
          @change="save('print_autopa', $event)"
          @click.native.stop
        />
      </app-setting>
    </template>
  </div>
</template>

<script lang="ts">
import { Component, Mixins } from 'vue-property-decorator'
import StateMixin from '@/mixins/state'

// Z-Mod: global print start flags in save_variables, as zmod_color writes them
@Component({})
export default class AfcPrintStartDialogZmod extends Mixins(StateMixin) {
  get printer () {
    return this.$typedState.printer.printer
  }

  get zmod (): boolean {
    return this.printer.zmod_color != null
  }

  get c5pro (): boolean {
    return this.printer['gcode_macro _CLIENT_VARIABLE']?.c5pro === true
  }

  get leveling (): boolean {
    return Number(this.printer.save_variables?.variables?.print_leveling) === 1
  }

  get autoPa (): boolean {
    return Number(this.printer.save_variables?.variables?.print_autopa) === 1
  }

  save (name: string, value: boolean) {
    this.sendGcode(`SAVE_VARIABLE VARIABLE=${name} VALUE=${value ? 1 : 0}`)
  }
}
</script>
