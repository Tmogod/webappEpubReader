<template>
  <div class="ebook" ref="ebook">
      <ebook-title></ebook-title>
      <ebook-reader></ebook-reader>
      <ebook-menu></ebook-menu>
      <ebook-book-mark></ebook-book-mark>
      <div class="ebook-patch-top"></div>
  </div>
</template>

<script>
import EbookReader from '../../components/ebook/EbookReader.vue'
import EbookTitle from '../../components/ebook/EbookTitle.vue'
import EbookMenu from '../../components/ebook/EbookMenu.vue'
import { ebookMixin } from '../../utils/mixin'
import { getReadTime, saveReadTime } from '../../utils/localStorage'
import EbookBookMark from '../../components/ebook/EbookBookMark.vue'
export default {
  mixins: [ebookMixin],
  components: { EbookReader, EbookTitle, EbookMenu, EbookBookMark },
  methods: {
    startLoopReadTime () {
      let readTime = getReadTime(this.fileName)
      if (!readTime) {
        readTime = 0
      }
      this.task = setInterval(() => {
        readTime++
        if (readTime % 30 === 0) {
          saveReadTime(this.fileName, readTime)
        }
      }, 1000)
    },
    move (v) {
      this.$refs.ebook.style.top = v + 'px'
    },
    restore () {
      this.$refs.ebook.style.top = 0 + 'px'
      this.$refs.ebook.style.transition = 'all .2s linear'
      setTimeout(() => {
        this.$refs.ebook.style.transition = ''
      }, 200)
    }
  },
  mounted () {
    this.startLoopReadTime()
  },
  beforeDestroy () {
    if (this.task) {
      clearInterval(this.task)
    }
  },
  watch: {
    offsetY (v) {
      if (!this.menuVisible && this.bookAvailable) {
        if (v > 0) {
          this.move(v)
        } else if (v === 0) {
          this.restore()
        }
      }
    }
  }
}
</script>

<style lang="scss" scoped>
  @import "../../assets/styles/global.scss";
  .ebook {
    width: 100%;
    height: 100%;
    position: absolute;
    top: 0;
    left: 0;
    .ebook-patch-top {
      width: 100%;
      height: 100%;
      position: absolute;
      left: 0;
      top: 0;
      transform: translateY(-100%);
      background: lightgray;
    }
  }
</style>
