<template>
  <div class="bookmark" :style="style" ref="bookmark" :width="15" :height="35"></div>
</template>

<script>
import { px2rem } from '../../utils/utils'
// const BLUE = '#346cbc'
// const WHITE = '#fff'
export default {
  props: {
    width: Number,
    height: Number,
    color: String
  },
  computed: {
    style () {
      if (this.color) {
        return {
          borderColor: `${this.color} ${this.color} transparent ${this.color}`
        }
      } else {
        return {
          borderColor: 'white white transparent white'
        }
      }
    }
  },
  methods: {
    refresh () {
      if (this.height && this.width) {
        this.$refs.bookmark.style.borderWidth = `${px2rem(this.height - 5)}rem ${px2rem(this.width / 2)}rem ${px2rem(5)}rem ${px2rem(this.width / 2)}rem`
        console.log(px2rem(2))
      }
    }
  },
  mounted () {
    this.refresh()
  }
}
</script>

<style lang="scss" rel="stylesheet/scss" scoped>
  @import "../../assets/styles/global";
        .bookmark {
        width: 0;
        height: 0;
        border-width: px2rem(20) px2rem(10) px2rem(10) px2rem(10);
        border-style: solid;
        border-color: white white transparent white;
      }
</style>
