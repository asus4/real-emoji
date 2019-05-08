<template lang="pug">
  .home
    .container(v-if="isLoading")
      h1 Loading ...
    .container(v-else)
      WebCam(ref="webcam" @loaded="onLoaded")
      canvas.overlay(ref="overlay")
      br
      .emoji {{ current }}
</template>


<script lang="ts">
import * as faceapi from 'face-api.js'
import { Component, Vue } from 'vue-property-decorator'
import WebCam from '@/components/WebCam.vue'

@Component({
  components: {
    WebCam,
  },
})
export default class Home extends Vue {

  public $refs!: {
    webcam: WebCam;
    overlay: HTMLCanvasElement;
  }

  public get video(): HTMLVideoElement { return this.$refs.webcam.video }
  public get overlay(): HTMLCanvasElement { return this.$refs.overlay }

  private isLoading = true
  private options = new faceapi.TinyFaceDetectorOptions()
  private emoji: { [key: string]: string; } = {
    neutral:  '😐',
    happy: '😀',
    sad: '😥',
    angry: '😤',
    fearful: '😖',
    disgusted: '🤮',
    surprised: '😲',
  }
  private current = '😐'

  private async mounted() {
    const modelpath = 'https://justadudewhohacks.github.io/face-api.js/models'
    await faceapi.loadTinyFaceDetectorModel(modelpath)
    await faceapi.loadFaceExpressionModel(modelpath)

    this.isLoading = false
  }

  private onLoaded() {
    this.update()
  }

  private async update() {
    await this.detect()
    requestAnimationFrame(this.update)
  }

  private async detect() {
    const video = this.video
    const canvas = this.overlay


    const result = await faceapi.detectSingleFace(video, this.options).withFaceExpressions()
    if (!result) {
      return
    }

    const dims = faceapi.matchDimensions(canvas, video, true)
    const resized = faceapi.resizeResults(result, dims)

    faceapi.draw.drawDetections(canvas, resized)
    faceapi.draw.drawFaceExpressions(canvas, resized, 0.05)

    const expressions = resized.expressions.asSortedArray()
    const current = expressions[0].expression
    this.current = this.emoji[current]
  }
}
</script>

<style lang="sass" scoped>
.container
  position: relative
  width: 640px
  height: 480px

.overlay
  position: absolute
  top: 0
  left: 0
  width: 100%
  height: 100%

.emoji
  display: block
  font-size: 120px

</style>

