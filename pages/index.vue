<template>
  <div>
    <b-container class="mt-3">
      <div class="card">
        <div class="card-body">
          <b-row>
            <b-col>
              <canvas
                ref="emojiCanvas"
                :width="canvasSize"
                :height="canvasSize"
                style="background-color: yellow"
              >
                このブラウザは HTML5 Canvas に対応していません。
              </canvas>
            </b-col>
          </b-row>
          <b-row>
            <b-col>
              <b-form-textarea
                v-model="emojiHandle"
                class="emoji-textarea"
                size="lg"
                rows="3"
                placeholder="文字入力"
              ></b-form-textarea>
            </b-col>
            <b-col>
              <b-list-group>
                <b-list-group-item
                  v-for="font in fonts"
                  :key="font.cssFontFamily"
                  ><b-button
                    block
                    variant="info"
                    @click="drawText(emojiHandle, font)"
                    >{{ font.cssFontFamily }}</b-button
                  ></b-list-group-item
                >
              </b-list-group>
            </b-col>
            <b-col>3 of 3</b-col>
          </b-row>
        </div>
      </div>
    </b-container>
  </div>
</template>

<script>
export default {
  data() {
    return {
      canvasSize: 300,
      emojiCanvasCtx: null,
      emojiHandle: '絵文字',
      fontLoader: null,
      fonts: {
        notoSansJp: {
          cssFontFamily: 'Noto Sans JP',
          loaderFontFamily: 'Noto+Sans+JP:700',
        },
        mPlusRounded1c: {
          cssFontFamily: 'M PLUS Rounded 1c',
          loaderFontFamily: 'M+PLUS+Rounded+1c:700',
        },
        pottaOne: {
          cssFontFamily: 'Potta One',
          loaderFontFamily: 'Potta+One:400',
        },
        yuseiMagic: {
          cssFontFamily: 'Yusei Magic',
          loaderFontFamily: 'Yusei+Magic:400',
        },
        hachiMaruPop: {
          cssFontFamily: 'Hachi Maru Pop',
          loaderFontFamily: 'Hachi+Maru+Pop:400',
        },
      },
    }
  },
  // watch: {
  //   emojiHandle(text) {
  //     this.drawText(text)
  //   },
  // },
  mounted() {
    this.emojiCanvasCtx = this.$refs.emojiCanvas.getContext('2d')
    this.drawText(this.emojiHandle, this.fonts.notoSansJp)
  },
  methods: {
    drawCanvas(rawText, fontData) {
      this.emojiCanvasCtx.clearRect(
        0,
        0,
        this.emojiCanvasCtx.canvas.clientWidth,
        this.emojiCanvasCtx.canvas.clientHeight
      )

      this.emojiCanvasCtx.textAlign = 'center'
      this.emojiCanvasCtx.textBaseline = 'middle'
      this.emojiCanvasCtx.fillStyle = 'rgba(0, 0, 255)'

      const textArray = rawText.split('\n')
      const rows = textArray.length
      const fontTargetSize = 296 / rows
      const canvasSize = this.canvasSize
      const emojiCanvasCtx = this.emojiCanvasCtx
      textArray.forEach(function (text, index) {
        emojiCanvasCtx.font =
          String(fontTargetSize) + "px '" + fontData.cssFontFamily + "'"
        const beforeMeasureResult = emojiCanvasCtx.measureText(String(text))
        const beforeHeight =
          beforeMeasureResult.actualBoundingBoxAscent +
          beforeMeasureResult.actualBoundingBoxDescent
        const diameter = fontTargetSize / beforeHeight
        emojiCanvasCtx.font =
          String(fontTargetSize * diameter) +
          "px '" +
          fontData.cssFontFamily +
          "'"
        const afterMeasureResult = emojiCanvasCtx.measureText(String(text))
        const offset =
          Math.abs(afterMeasureResult.actualBoundingBoxAscent) -
          Math.abs(
            afterMeasureResult.actualBoundingBoxAscent +
              afterMeasureResult.actualBoundingBoxDescent
          ) /
            2

        const yPos = canvasSize / rows / 2 + (canvasSize / rows) * index
        emojiCanvasCtx.fillText(
          String(text),
          canvasSize / 2,
          yPos + offset,
          300
        )
      })
    },
    fontLoaderInit() {
      if (this.fontLoader === null) this.fontLoader = require('webfontloader')
    },
    drawText(rawText, fontData) {
      this.fontLoaderInit()
      const instance = this
      const config = {
        google: {
          families: [fontData.loaderFontFamily],
          text: rawText,
        },
        active() {
          instance.drawCanvas(rawText, fontData)
        },
        classes: false,
      }
      this.fontLoader.load(config)
    },
  },
}
</script>

<style lang="scss" scoped>
.emoji-textarea {
  //height: 20rem;
  width: 10rem;
  font-size: 1.8rem;
}
</style>
