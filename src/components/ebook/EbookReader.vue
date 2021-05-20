<template>
  <div class="ebook-reader">
      <div id="read"></div>
  </div>
</template>

<script>
import { ebookMixin } from '../../utils/mixin'
import Epub from 'epubjs'
import { getFontFamily, saveFontFamily, getFontSize, saveFontSize, getTheme, saveTheme, getLocation } from '../../utils/localStorage'
global.ePub = Epub
export default {
  mixins: [ebookMixin],
  methods: {
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
      console.log(defaultTheme, '1111')
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
          console.log('字体全部加载完')
        })
      })
    },
    initGesture () {
      this.rendition.on('touchstart', event => {
        console.log(event)
        this.touchStartX = event.changedTouches[0].clientX
        this.touchStartTime = event.timeStamp
      })
      this.rendition.on('touchend', event => {
        const offsetX = event.changedTouches[0].clientX - this.touchStartX
        const time = event.timeStamp - this.touchStartTime
        console.log(offsetX, time)
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
    initEpub () {
      const url = 'http://1.116.178.71/epub/' + this.fileName
      // const url = 'http://127.0.0.1:8989/epub/' + this.fileName
      this.book = new Epub(url)
      this.setCurrentBook(this.book)
      this.initRendition()
      this.initGesture()
      this.book.ready.then(() => {
        return this.book.locations.generate(750 * (window.innerWidth / 375) * (getFontSize(this.fileName) / 16)).then(locations => {
          this.setBookAvailable(true)
          this.refreshLocation()
        })
      })
    }
  },
  mounted () {
    console.log(this.$route.params.filename.split('|').join('/'))
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
    @import "../../assets/styles/global"
</style>
