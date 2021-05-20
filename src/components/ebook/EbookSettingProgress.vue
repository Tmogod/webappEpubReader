<template>
  <transition name="slide-up">
    <div class="setting-wrapper"  v-show="menuVisible && settingVisible === 2">
      <div class="setting-progress">
        <div class="read-time-wrapper">
          <span class="read-time-text">{{getReadTimeText()}}</span>
          <span class="icon-forward"></span>
        </div>
        <div class="progress-wrapper">
          <div class="progress-icon-wrapper" @click="preSection()">
            <span class="icon-back"></span>
          </div>
          <input class="progress" type="range"
                 max="100"
                 min="0"
                 step="1"
                 @change="onProgressChange($event.target.value)"
                 @input="onProgressInput($event.target.value)"
                 :value="progress"
                 :disabled="!bookAvailable"
                 ref="progress">
          <div class="progress-icon-wrapper" @click="nextSection()">
            <span class="icon-forward"></span>
          </div>
        </div>
        <div class="text-wrapper">
          <span class="progress-section-text">{{getSectionName}}</span>
          <span>{{bookAvailable ? progress + '%' : '加载中...'}}</span>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
import { getReadTimeByMinute } from '../../utils/book.js'
// import { getReadTime } from '../../utils/localStorage.js'
import { ebookMixin } from '../../utils/mixin.js'
// import { saveLocation } from '../../utils/localStorage'

export default {
  mixins: [ebookMixin],
  computed: {
    getSectionName () {
      if (this.section) {
        const sectionInfo = this.currentBook.section(this.section)
        if (sectionInfo && sectionInfo.href) {
          return this.currentBook.navigation.get(sectionInfo.href).label
        }
      }
      return ''
    }
  },
  methods: {
    getReadTimeText () {
      return this.$t('book.haveRead').replace('$1', getReadTimeByMinute(this.fileName))
    },
    onProgressChange (progress) {
      this.setProgress(progress).then(() => {
        this.displayProgress()
      })
    },
    onProgressInput (progress) {
      this.setProgress(progress)
      this.updateProgressBg(progress)
    },
    displayProgress () {
      const cfi = this.currentBook.locations.cfiFromPercentage(this.progress / 100)
      this.currentBook.rendition.display(cfi).then(() => {
        this.refreshLocation()
      })
    },
    updateProgressBg (progress) {
      this.$refs.progress.style.backgroundSize = `${progress}% 100%`
    },
    preSection () {
      console.log('aaaa')
      if (this.section > 0 && this.bookAvailable) {
        this.setSection(this.section - 1).then(() => {
          this.displaySection()
        })
      }
    },
    nextSection () {
      console.log(this.currentBook.spine)
      if (this.section < this.currentBook.spine.length - 1 && this.bookAvailable) {
        this.setSection(this.section + 1).then(() => {
          this.displaySection()
        })
      }
    },
    displaySection () {
      const sectionInfo = this.currentBook.section(this.section)
      if (sectionInfo && sectionInfo.href) {
        this.currentBook.rendition.display(sectionInfo.href).then(() => {
          this.refreshLocation()
        })
      }
    }
  },
  updated () {
    this.updateProgressBg(this.progress)
  }
}
</script>

<style lang="scss" scoped>
  @import '../../assets/styles/global.scss';
  .setting-wrapper {
    position: absolute;
    bottom: px2rem(48);
    left: 0;
    z-index: 101;
    width: 100%;
    height: px2rem(60);
    background: white;
    box-shadow: 0 px2rem(-8) px2rem(8) rgba(0, 0, 0, .15);
    .setting-progress {
        position: relative;
        width: 100%;
        height: 100%;
        .read-time-wrapper {
          position: absolute;
          left: 0;
          top: 0;
          width: 100%;
          height: px2rem(20);
          font-size: px2rem(10);
          @include center;
        }
        .progress-wrapper {
          width: 100%;
          height: 100%;
          @include center;
          padding: 0 px2rem(15);
          box-sizing: border-box;
          .progress-icon-wrapper {
            font-size: px2rem(20);
          }
          .progress {
            width: 100%;
            -webkit-appearance: none;
            height: px2rem(2);
            /* background: -webkit-linear-gradient(#999, #999) no-repeat, #ddd; */
            background: -webkit-linear-gradient(white, white) no-repeat, rgb(139, 133, 133);
            /* background-size: 0 100%!important; */
            background-size: 75% 100%;
            margin: 0 px2rem(10);
            &:focus {
              outline: none;
            }
            &::-webkit-slider-thumb {
              -webkit-appearance: none;
              height: px2rem(20);
              width: px2rem(20);
              border-radius: 50%;
              background: white;
              box-shadow: 0 4px 4px 0 rgba(0, 0, 0, .15);
              border: px2rem(1) solid #ddd;
            }
          }
        }
        .text-wrapper {
          position: absolute;
          left: 0;
          bottom: px2rem(10);
          width: 100%;
          color: #333;
          font-size: px2rem(10);
          text-align: center;
          white-space: nowrap;
          overflow: hidden;
        }
      }
  }
</style>
