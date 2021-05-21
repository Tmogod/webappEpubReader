<template>
  <div class="ebook-reader">
      <div id="read"></div>
      <div class="ebook-reader-mask"
      @click="onMaskClick"
      @touchmove="move"
      @touchend="moveEnd"></div>
  </div>
</template>

<script>
import { ebookMixin } from '../../utils/mixin'
import Epub from 'epubjs'
import { getFontFamily, saveFontFamily, getFontSize, saveFontSize, getTheme, saveTheme, getLocation } from '../../utils/localStorage'
import { flatten } from '../../utils/book'
global.ePub = Epub
export default {
  mixins: [ebookMixin],
  methods: {
    move (e) {
      let offsetY = 0
      if (this.firstOffsetY) {
        offsetY = e.changedTouches[0].clientY - this.firstOffsetY

        this.setOffsetY(offsetY)
      } else {
        this.firstOffsetY = e.changedTouches[0].clientY
      }
      e.preventDefault()
      e.stopPropagation()
    },
    moveEnd (e) {
      this.setOffsetY(0)
      this.firstOffsetY = null
    },
    onMaskClick (e) {
      const offsetX = e.offsetX
      const width = window.innerWidth
      if (offsetX > 0 && offsetX < width * 0.3) {
        this.prevPage()
      } else if (offsetX > 0 && offsetX > width * 0.7) {
        this.nextPage()
      } else {
        this.toggleTitleAndMenu()
      }
    },
    prevPage () {
      if (this.rendition) {
        this.rendition.prev().then(() => {
          this.refreshLocation()
        })
        this.hideTitleAndMenu()
      }
    },
    nextPage () {
      if (this.rendition) {
        this.rendition.next().then(() => {
          this.refreshLocation()
        })
        this.hideTitleAndMenu()
      }
    },
    toggleTitleAndMenu () {
      // console.log('111', this.menuVisible)
      // this.$store.dispatch('setMenuVisible', !this.menuVisible)
      this.setMenuVisible(!this.menuVisible)
      if (this.menuVisible === true) {
        this.setSettingVisible(-1)
      }
      this.setFontFamilyVisible(false)
    },
    hideTitleAndMenu () {
      // this.$store.dispatch('setMenuVisible', false)
      this.setMenuVisible(false)
      this.setSettingVisible(-1)
      this.setFontFamilyVisible(false)
    },
    intFontFamily () {
      const font = getFontFamily(this.fileName)
      if (!font) {
        saveFontFamily(this.fileName, this.defaultFontFamily)
      } else {
        this.rendition.themes.font(font)
        this.setDefaultFontFamily(font)
      }
    },
    initFontSize () {
      const fontSize = getFontSize(this.fileName)
      if (fontSize == null) {
        saveFontSize(this.fileName, this.defaultFontSize)
      } else {
        this.setDefaultFontSize(fontSize)
        this.rendition.themes.font(fontSize)
      }
    },
    intiTheme () {
      let defaultTheme = getTheme(this.fileName)
      if (defaultTheme == null) {
        defaultTheme = this.themeList[0].name
        saveTheme(this.filename, defaultTheme)
      }
      this.themeList.forEach(theme => {
        this.rendition.themes.register(theme.name, theme.style)
      })
      this.rendition.themes.select(defaultTheme)
      this.setDefaultTheme(defaultTheme)
    },
    display (target, cb) {
      if (target) {
        return this.currentBook.rendition.display(target).then(() => {
          this.refreshLocation()
          if (cb) cb()
        })
      } else {
        return this.currentBook.rendition.display().then(() => {
          this.refreshLocation()
          if (cb) cb()
        })
      }
    },
    initRendition () {
      this.rendition = this.book.renderTo('read', {
        width: innerWidth,
        height: innerHeight,
        method: 'default'
      })
      const location = getLocation(this.fileName)
      this.display(location, () => {
        this.intFontFamily()
        this.initFontSize()
        this.intiTheme()
        this.initGlobalStyle()
        this.refreshLocation()
      })
      this.rendition.hooks.content.register(contents => {
        Promise.all([
          contents.addStylesheet('http://1.116.178.71/font/daysOne.css'),
          contents.addStylesheet('http://1.116.178.71/font/cabin.css'),
          contents.addStylesheet('http://1.116.178.71/font/montserrat.css'),
          contents.addStylesheet('http://1.116.178.71/font/tangerine.css')
        ]).then(() => {
        })
      })
    },
    initGesture () {
      this.rendition.on('touchstart', event => {
        this.touchStartX = event.changedTouches[0].clientX
        this.touchStartTime = event.timeStamp
      })
      this.rendition.on('touchend', event => {
        const offsetX = event.changedTouches[0].clientX - this.touchStartX
        const time = event.timeStamp - this.touchStartTime
        if (time < 500 && offsetX > 40) {
          this.prevPage()
        } else if (time < 500 && offsetX < -40) {
          this.nextPage()
        } else {
          this.toggleTitleAndMenu()
        }
        event.preventDefault()
        event.stopPropagation()
      })
    },
    parseBook () {
      this.currentBook.loaded.cover.then(cover => {
        this.book.archive.createUrl(cover).then(url => {
          this.setCover(url)
          // console.log(url)
        })
      })
      this.book.loaded.metadata.then(metadata => {
        this.setMetadata(metadata)
      })
      this.book.loaded.navigation.then(nav => {
        const navItem = flatten(nav.toc)
        function find (item, level = 0) {
          return !item.parent ? level : find(navItem.filter(parentItem => parentItem.id === item.parent)[0], ++level)
        }
        navItem.forEach(item => {
          item.level = find(item)
        })
        this.setNavigation(navItem)
      })
    },
    initEpub () {
      const url = 'http://1.116.178.71/epub/' + this.fileName
      // const url = 'http://127.0.0.1:8989/epub/' + this.fileName
      this.book = new Epub(url)
      this.setCurrentBook(this.book)
      this.initRendition()
      this.initGesture()
      this.parseBook()
      this.book.ready.then(() => {
        return this.book.locations.generate(750 * (window.innerWidth / 375) * (getFontSize(this.fileName) / 16)).then(locations => {
          this.setBookAvailable(true)
          this.refreshLocation()
        })
      })
    }
  },
  mounted () {
    this.setFileName(this.$route.params.filename.split('|').join('/') + '.epub').then(() => {
      this.initEpub()
    })
    // console.log(this.setFileName(this.$route.params.filename.split('|').join('/') + '.epub'))
  },
  components: {
  }
}
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
    @import "../../assets/styles/global";
    .ebook-reader {
      width: 100%;
      height: 100%;
      overflow: hidden;
      .ebook-reader-mask {
        position: absolute;
        z-index: 99;
        top: 0;
        left: 0;
        background: transparent;
        width: 100%;
        height: 100%;
      }
    }
</style>
