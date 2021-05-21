<template>
  <div class="ebook-bookmark" ref="bookmark">
    <div class="ebook-bookmark-text-wrapper">
      <div class="ebook-bookmark-down-wrapper">
        <span class="icon-down" ref="iconDown"></span>
      </div>
      <div class="ebook-bookmark-text">{{text}}</div>
    </div>
    <div class="ebook-bookmark-icon-wrapper" :style="isFixed? fixedStyle : {}">
      <div class="icon">
        <bookmark :color="color" :width="15" :height="35"></bookmark>
      </div>
    </div>
  </div>
</template>

<script>
import Bookmark from '../common/Bookmark.vue'
import { realPx, px2rem } from '../../utils/utils'
import { ebookMixin } from '../../utils/mixin'
import { getBookmark, saveBookmark } from '../../utils/localStorage'

const BLUE = '#346cbc'
const WHITE = '#fff'
export default {
  mixins: [ebookMixin],
  components: { Bookmark },
  data () {
    return {
      text: '',
      color: WHITE,
      isFixed: false
    }
  },
  computed: {
    height () {
      return realPx(35)
    },
    threshold () {
      return realPx(55)
    },
    fixedStyle () {
      return {
        position: 'fixed',
        right: `${(window.innerWidth - this.$refs.bookmark.clientWidth) / 2}px`,
        top: 0
      }
    }
  },
  watch: {
    isBookmark (isBookmark) {
      if (isBookmark) {
        this.color = BLUE
        this.isFixed = true
      } else {
        this.color = WHITE
        this.isFixed = false
      }
    },
    offsetY (v) {
      if (!this.bookAvailable || this.menuVisible || this.settingVisible >= 0) {
        return
      }
      if (v >= this.height && v < this.threshold) {
        this.beforeThreshold(v)
      } else if (v >= this.threshold) {
        this.afterThreshold(v)
      } else if (v > 0 && v < this.height) {
        this.beforeHeight()
      }

      const iconDown = this.$refs.iconDown
      if (v === 0) {
        setTimeout(() => {
          this.$refs.bookmark.style.top = `-${px2rem(35)}rem`
          iconDown.style.transform = ''
          if (this.isFixed) {
            this.color = BLUE
            this.setIsBookmark(true)
            this.addBookmark()
          } else {
            this.color = WHITE
            this.setIsBookmark(false)
            this.removeBookmark()
          }
        }, 200)
      }
    }
  },
  methods: {
    addBookmark () {
      this.bookmark = getBookmark(this.fileName)
      if (!this.bookmark) {
        this.bookmark = []
      }
      const currentLocation = this.currentBook.rendition.currentLocation()
      const cfibase = currentLocation.start.cfi.replace(/!.*/, '')
      const cfistart = currentLocation.start.cfi.replace(/.*!/, '').replace(/\)/, '')
      const cfiend = currentLocation.end.cfi.replace(/.*!/, '').replace(/\)/, '')
      console.log(cfibase, cfistart, cfiend)
      const cfirange = `${cfibase}!,${cfistart},${cfiend})`
      this.currentBook.getRange(cfirange).then(range => {
        const text = range.toString().replace(/\s\s/g, '')
        this.bookmark.push({
          cfi: currentLocation.start.cfi,
          text: text
        })
        saveBookmark(this.fileName, this.bookmark)
      })
    },
    removeBookmark () {
      const currentLocation = this.currentBook.rendition.currentLocation()
      const cfi = currentLocation.start.cfi
      this.bookmark = getBookmark(this.fileName)
      if (this.bookmark) {
        this.bookmark = this.bookmark.filter(item => item.cfi !== cfi)
        saveBookmark(this.fileName, this.bookmark)
        this.setIsBookmark(false)
      }
    },
    beforeHeight () {
      if (this.isBookmark) {
        this.text = this.$t('book.releaseAddMark')
        this.color = BLUE
        this.isFixed = true
      } else {
        this.text = this.$t('book.pulldownAddMark')
        this.color = WHITE
        this.isFixed = false
      }
    },
    beforeThreshold (v) {
      // 状态2：未到达零界状态
      this.$refs.bookmark.style.top = `${-v}px`
      this.beforeHeight()
    },
    afterThreshold (v) {
      const iconDown = this.$refs.iconDown
      this.$refs.bookmark.style.top = `${-v}px`
      if (this.isBookmark) {
        this.text = this.$t('book.pulldownAddMark')
        this.color = WHITE
        this.isFixed = false
      } else {
        this.text = this.$t('book.releaseAddMark')
        this.color = BLUE
        this.isFixed = true
      }
      if (iconDown.style.transform === '') {
        iconDown.style.transform = 'rotate(180deg)'
      }
    }

  }
}
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
  @import "../../assets/styles/global";
  .ebook-bookmark {
    position: absolute;
    top: px2rem(-35);
    left: 0;
    z-index: 200;
    height: px2rem(35);
    width: 100%;
    background: lightgray;
    .ebook-bookmark-text-wrapper {
      position: absolute;
      right: px2rem(45);
      bottom: 0;
      display: flex;
      .ebook-bookmark-down-wrapper {
        font-size: px2rem(14);
        color: white;
        transition: all .2s linear;
        @include center;
      }
      .ebook-bookmark-text {
        font-size: px2rem(14);
        color: white
      }
    }
    .ebook-bookmark-icon-wrapper {
      position: absolute;
      right: 0;
      bottom: 0;
      margin-right: px2rem(15);
    }
  }
</style>
