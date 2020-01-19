<template>
    <div :class="['mui-marquee',marqueeClass]" :style="boxStyle">
        <ul class="mui-marquee__box" ref="box" :style="contentStyle">
          <li v-for="(item,index) in list" :key="item.id || index" v-text="itemTxt(item)"></li>
        </ul>
    </div>
</template>
<script>
/**
 * mui-marquee
 * @module components/marquee
 * @desc 跑马灯
 * @param {String} [direction] - 滚动方向 up down horizontal horizontal_interval
 * @param {Number} [showLength] - 展示区域个数
 * @param {Number} [speed] - 滚动速度,单位, eg: 1000/s
 * @param {Number} [interval] - 停留时间,默认为0,不停留
 * @param {Number} [listHeight] - 列表单项高度
 * @param {Number} [marqueeWid] - 跑马灯整体宽度
 * @param {Number} [scrollNum] - 每次滚动条数，只在间隔情况下有效
 */
import { setTimeout } from 'timers'
export default {
  name: 'm-bmarquee',
  props: {
    list: {
      type: Array,
      default: () => {
        return []
      }
    },
    liTemp: {
      type: String,
      default: ''
    },
    direction: {
      type: String,
      default: 'horizontal',
      validator: (value) => {
        return value === 'up' || value === 'down' || value === 'horizontal'
      }
    },
    showLength: {
      type: Number,
      default: 1,
      validator: (value) => {
        let reg = /^\d+$/
        return reg.test(value)
      }
    },
    speed: {
      type: Number,
      default: 20,
      validator: (value) => {
        let reg = /^\d+$/
        return reg.test(value)
      }
    },
    interval: {
      type: Number,
      default: 0,
      validator: (value) => {
        let reg = /^\d+$/
        return reg.test(value)
      }
    },
    listHeight: {
      type: Number,
      default: 40,
      validator: (value) => {
        let reg = /^\d+$/
        return reg.test(value)
      }
    },
    // marqueeWid: {
    //   type: Number,
    //   validator: (value) => {
    //     let reg = /^\d+$/
    //     return reg.test(value)
    //   }
    // },
    scrollNum: {
      type: Number,
      default: 1,
      validator: (value) => {
        let reg = /^\d+$/
        return reg.test(value)
      }
    }
  },
  data() {
    return {
      timer: '',
      scroll_base: 0,
      requestAnimFrameId: '',
      cloneNode: '',
      base_count: 0
    }
  },
  computed: {
    listHeightRem() {
      return this.toRem(this.listHeight)
    },
    marqueeClass() {
      return 'mui-marquee-' + this.direction
    },
    // 换算16.7ms移动的距离
    perMs() {
      return this.speed / 1000 * 16.7
    },
    boxStyle() {
      return {
        'height': this.listHeightRem * this.showLength + 'rem'
      }
    },
    contentStyle() {
      if (this.direction === 'horizontal') {
        return {
          'line-height': this.listHeightRem + 'rem',
          'height': this.listHeightRem + 'rem'
        }
      } else {
        return {
          'line-height': this.listHeightRem + 'rem'
        }
      }
    }
  },
  created() {
    window.requestAnimFrame = (function() {
      return window.requestAnimationFrame ||
          window.webkitRequestAnimationFrame ||
          window.mozRequestAnimationFrame ||
          function(callback) {
            window.setTimeout(callback, 1000 / 60)
          }
    })()
  },
  mounted() {
    this.init()
  },
  methods: {
    // 清除动画
    destroy() {
      this.requestAnimFrameId && window.cancelAnimationFrame(this.requestAnimFrameId)
    },
    init() {
      this.destroy()
      // 横向间隔重置li宽度
      if (this.direction === 'horizontal' && this.interval) this.formatLiWidth()

      // 调用动画
      this.requestAnimFrameId = window.requestAnimFrame(this.commonFn)
    },
    formatLiWidth() {
      // let boxW = this.$refs.box.parentNode.parentNode.offsetWidth
      let boxW = this.$refs.box.parentNode.offsetWidth
      for (let i = 0; i < this.$refs.box.childNodes.length; i++) {
        this.$refs.box.childNodes[i].style.width = boxW + 'px'
      }
    },
    // 抽取通用事件
    commonFn() {
      let box = this.$refs.box
      let boxH = this.$refs.box.offsetHeight
      let firstItem = box.firstElementChild
      let lastItem = box.lastElementChild
      let reference = this.direction === 'horizontal' ? firstItem.offsetWidth : this.listHeight
      this.scroll_base += this.perMs
      if (this.direction === 'horizontal') {
        box.style.transform = 'translate3d(' + (-this.scroll_base) + 'px,0,0)'
      } else if (this.direction === 'up') {
        box.style.transform = 'translate3d(0,' + (-this.toRem(this.scroll_base)) + 'rem,0)'
      } else if (this.direction === 'down') {
        let y = this.toRem(boxH - this.listHeight * this.showLength - this.scroll_base).toFixed(4)
        // let y = boxH - this.listHeight * this.showLength - this.scroll_base
        box.style.transform = 'translate3d(0,' + -y + 'rem,0)'
      }

      if (this.scroll_base >= reference) {
        this.base_count++
        // 无间隔不销毁动画
        this.interval && this.destroy()

        // 克隆添加到对应位置
        this.cloneNode = null
        let readyItem = this.direction === 'down' ? lastItem : firstItem
        this.cloneNode = readyItem.cloneNode(true)
        if (this.direction === 'down') {
          box.insertBefore(this.cloneNode, box.childNodes[0])
        } else {
          box.appendChild(this.cloneNode)
        }

        // 删除已滚动条目，保持数据正确
        box.removeChild(readyItem)

        // 重置基数
        this.scroll_base = 0

        // 重置list位置，恢复初始位置
        if (this.direction === 'down') {
          let y = this.toRem(boxH - this.listHeight * this.showLength).toFixed(4)
          this.$refs.box.style.transform = 'translate3d(0,' + -y + 'rem,0)'
        } else {
          this.$refs.box.style.transform = 'translate3d(0,0,0)'
        }
        // 判断连续or间隔
        // console.log(this.interval, this.base_count)
        if (this.interval && this.base_count === this.scrollNum) {
          this.base_count = 0
          setTimeout(() => {
            this.requestAnimFrameId = window.requestAnimFrame(this.commonFn)
          }, this.interval)
        } else {
          this.requestAnimFrameId = window.requestAnimFrame(this.commonFn)
        }
      } else {
        this.requestAnimFrameId = window.requestAnimFrame(this.commonFn)
      }
    },
    toRem(val) {
      return window.hotcss.px2rem(val, 750)
    },
    itemTxt(item) {
      // 没传，默认认为是字符串数组，直接取值
      if (!this.liTemp) return item
      // 有模板，却没有变量，返回模板内容
      if (this.liTemp && !this.liTemp.includes('$')) return this.liTemp

      return this.parseTemp(item)
    },
    parseTemp(item) {
      let reg = /\$[a-zA-Z]+/g
      let text = this.liTemp.replace(reg, (match) => {
        let _key = match.split('$')[1]
        return item[_key]
      })
      return text
    }
  }
}
</script>
<style lang="scss">
@import './style.scss'
</style>
