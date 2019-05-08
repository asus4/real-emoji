<template lang="pug">
  video.webcam(
    ref="video" autoplay muted
    @loadeddata="onLoaded"
  )
</template>


<script lang="ts">
import { Component, Vue, Emit } from 'vue-property-decorator'

@Component({
  components: {
  },
})
export default class WebCam extends Vue {

  public $refs!: {
    video: HTMLVideoElement;
  }

  public get video(): HTMLVideoElement { return this.$refs.video }

  private async mounted() {
    const stream = await navigator.mediaDevices.getUserMedia({ video: {} })

    this.video.srcObject = stream
  }

  @Emit('loaded')
  private onLoaded() {
    return
  }
}
</script>
