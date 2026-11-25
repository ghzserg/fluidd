<template>
  <v-card
    v-if="camera"
    class="overflow-hidden"
  >
    <CameraItem
      :camera="camera"
      fullscreen
    />
  </v-card>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import CameraItem from '@/components/widgets/camera/CameraItem.vue'

@Component({
  components: {
    CameraItem
  }
})
export default class FullscreenCamera extends Vue {
  get camera (): Moonraker.Webcam.Entry | null {
    return this.$typedGetters['webcams/getWebcamById'](this.$route.params.cameraId) ?? null
  }
}
</script>
