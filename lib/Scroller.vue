<template>
  <div id="app"
  	   class="mod-scroller"
       @touchmove="touchMove"
       @touchend="touchEnd">
    <div class="refresh" v-show="!loading && topShow">{{topText}}</div>
    <iscroll-view
      ref="scrollView"
      class="scroll-view"
      :options="options"
      :style="styles"
      @beforeScrollStart="beforeScrollStart">
      <div class="scroller-slot" ref="scrollerSlot">
        <slot></slot>
        <div class="nomore-box" v-show="noMore">
          <div class="nomore">{{noMoreText}}</div>
        </div>
      </div>
    </iscroll-view>
    <div class="loadmore" v-show="!loading && !noMore && bottomShow">{{bottomText}}</div>
  </div>
</template>

<script type="text/ecmascript-6">
  import Vue from 'vue'
  import IScrollView from 'vue-iscroll-view'
  import IScroll from 'iscroll'

  Vue.use(IScrollView, IScroll)
  export default {
    components: {},
    data() {
      return {
        iTop: 0,
        iHeight: 50,
        topShow: false,      // 按下时显示默认头部
        bottomShow: false,   // 按下时显示默认底部
        topEnough: false,    // 头部拖动了足够的高度
        bottomEnough: false, // 底部拖动了足够的高度
        noMore: false,       // 没有更多
        topText: '下拉刷新',
        bottomText: '上拉加载更多',
        noMoreText: '没有更多了',
        refreshFlag: false,   // 刷新中状态
        loadmoreFlag: false,  // 加载中状态
        maxScrollY: 0,        // 最大可拖动距离
        options: {
          preventDefault: true,
          mouseWheel: true
        }
      }
    },
    props: {
      loading: {
        type: Number || Boolean,
        required: true
      }
    },
    computed: {
      styles() {
        const t = `translate(0px, ${this.iTop}px) translateZ(0px)`
        let styles = {
          '-webkit-transform': t,
          'transform': t,
          'transition-timing-function': 'cubic-bezier(0.1, 0.57, 0.1, 1)',
          'transition-duration': '600ms'
      }
        return styles
      }
    },
    methods: {
      /* 移动前获取最大滚动边界 */
      beforeScrollStart(iscroll) {
        this.maxScrollY = iscroll.maxScrollY
      },
      /* 判断移动的是scroller */
      isMyScroll(e) {
        let isScroller = false
        if (e && e.path) {
          e.path.forEach(item => {
            if (item.className === 'scroll-view') {
              isScroller = true
            }
          })
        } else {
          isScroller = true
        }
        return isScroller
      },
      /* 拖动事件 */
      touchMove(e) {
        if (this.refreshFlag || this.loadmoreFlag) return
        // transform: translate(0px, -784.677px) translateZ(0px)
        let obj = document.querySelector('.scroll-view > div')
        let iTransform = obj.style['-webkit-transform'] || obj.style['transform']
        let translateY = iTransform.split(' ')[1].split('px')[0]

        if (+translateY > 0) {
          this.topShow = true
          this.topText = '下拉刷新'
          if (+translateY > this.iHeight) {
            this.topEnough = true
            this.topText = '松开以刷新'
          }
        }
        if ((+translateY < 0) && !this.noMore) {
          this.bottomShow = true
          this.bottomText = '上拉加载更多'
          if ((+translateY < (this.maxScrollY - this.iHeight)) && !this.noMore) {
            this.bottomEnough = true
            this.bottomText = '松开以加载'
          }
        }
        e.preventDefault()
      },
      /* 拖动结束事件 */
      touchEnd(e) {
        this.topShow = false
        this.bottomShow = false
        if (this.topEnough) {
          this.topShow = true
          this.iTop = this.iHeight
          this.doRefresh()
        }
        if (this.bottomEnough && !this.noMore) {
          this.bottomShow = true
          this.iTop = -(this.iHeight)
          this.doLoadmore()
        }
        if (this.noMore) {
          this.bottomShow = true
        }
      },
      /* 刷新事件 */
      doRefresh(iscroll) {
        if (this.topEnough && !this.refreshFlag) {
          this.refreshFlag = true
          this.topText = '刷新中...'
          this.$emit('onRefresh', this.refreshDone)
        }
      },
      /* 加载事件 */
      doLoadmore(iscroll) {
        if (this.bottomEnough && !this.loadmoreFlag) {
          this.loadmoreFlag = true
          this.bottomText = '加载中...'
          this.$emit('onLoadmore', this.loadmoreDone)
        }
      },
      /* 刷新完成 */
      refreshDone(noMore) {
        this.$refs.scrollView.refresh()
        this.topShow = false
        this.topEnough = false
        this.noMore = noMore
        this.refreshFlag = false
        this.topText = '刷新完成'
        this.iTop = 0
      },
      /* 加载完成 */
      loadmoreDone(noMore) {
        this.$refs.scrollView.refresh()
        this.bottomShow = false
        this.bottomEnough = false
        this.noMore = noMore
        this.loadmoreFlag = false
        this.bottomText = '加载完成'
        this.iTop = 0
        if (this.noMore) {
          const obj = document.querySelector('.scroll-view > div')
          let h = 0
          if(this.lessMinHeight()){
            h = 0
          } else {
            h = this.maxScrollY - this.iHeight
          }
          obj.style.transform = `translate(0px, ${h}px) translateZ(0px)`
        }
      },
      /* 若内容容器小于滚动容器，则给内容容器设置默认最小高度，用以拖动 */
      lessMinHeight(){
        let isLessThan = false
        const minHeight = document.querySelector('.mod-scroller').offsetHeight + 1
        if(this.$refs.scrollerSlot.offsetHeight <= minHeight){
          isLessThan = true
          this.$refs.scrollerSlot.style['min-height'] = minHeight + 'px'
        }
        return isLessThan
      }
    },
    mounted () {
      this.lessMinHeight()
    },
    watch: {
      loading(newVal, oldVal){
        if(!newVal){
          this.$refs.scrollView.refresh()
        }
      }
    }
  }
</script>

<style scoped>
  .mod-scroller {
  	height: 100%;
  	position: relative;
  	z-index: 1;
  }
  .mod-scroller .scroll-view {
  	touch-action: none;
  	position: absolute;
  	z-index: 3;
  	top: 0;
  	bottom: 0;
  	left: 0;
  	right: 0;
  	overflow: hidden;
  }
  .mod-scroller .scroll-view .scroller-slot {
  	position: relative;
  }
  .mod-scroller .scroll-view .scroller-slot .nomore-box {
  	width: 100%;
  	height: 50px;
  }
  
  .mod-scroller .scroll-view .scroller-slot .nomore-box .nomore {
  	position: absolute;
  	left: 0;
  	bottom: 0;
  }
  .mod-scroller .refresh,
  .mod-scroller .loadmore,
  .mod-scroller .nomore {
  	width: 100%;
  	height: 50px;
  	line-height: 50px;
  	text-align: center;
  }
  .mod-scroller .refresh {
  	position: absolute;
  	z-index: 2;
  	top: 0px;
  	left: 0;
  }
  .mod-scroller .loadmore {
  	position: absolute;
  	z-index: 2;
  	bottom: 0px;
  	left: 0;
  }
</style>
