# vue-scroller

## 基于vue2.0的上拉加载&下拉刷新组件

## 使用方法：  

  1. import Scroller from '../../common/__Scroller/index.vue'  
  2. components: {  
       Scroller  
     }  
  3. data() {  
       return {  
         noMore: false,  // 没有更多了  
         loading: true   // 页面加载的状态  
     }  
  4. methods: {  
      /* 触发刷新时的处理函数 */  
      onRefresh(done) {  
        this.noMore = false  
        // TODO ...  
        // 完成请求后，手动执行done()，关闭加载占位  
      },  
      /* 触发加载更多时的处理函数 */  
      onLoadmore(done) {  
        // TODO ...  
        // 完成请求后，手动执行done()，关闭加载占位  
        // 如没有更多了，需将 this.noMore 置为 true  
      }  
    }  
