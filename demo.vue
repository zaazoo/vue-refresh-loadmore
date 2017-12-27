<template>
  <div class="mod-wrap">
    <scroller
      v-if="!loading && list.length"
      :loading="loading"
      :noMore="noMore"
      @onRefresh="onRefresh"
      @onLoadmore="onLoadmore">
      <ul>
        <li v-for="(item, index) in list" :key="index">...</li>
      </ul>
    </scroller>
    <div class="no-data" v-if="!loading && !list.length">暂无数据</div>
    <Loading :loading="loading"></Loading>
  </div>
</template>

<script type="text/ecmascript-6">
  // scroller组件
  import Scroller from '../../common/__Scroller/index.vue'
  // 页面loading组件
  import Loading from '../../common/Loading.vue'
  // 数据接口
  import {__getList} from '../../../assets/js/api/_getData'

  export default {
    components: {
      Loading, Scroller
    },
    data() {
      return {
        loading: true,
        noMore: false,
        list: [],
        params: {
          page: 1
        }
      }
    },
    methods: {
      /* 触发刷新时的处理函数 */
      onRefresh(done) {
        this.noMore = false
        this.params.page = 1
        this.getData('refresh', done)
      },
      /* 触发加载更多时的处理函数 */
      onLoadmore(done) {
        this.params.page++
        this.getData('loadmore', done)
      },
      /* 获取数据 */
      getData(type, done) {
        // 页面初始化时，使用页面loading占位
        if (!type) {
          this.loading = 1
        }
        // 请求数据
        __getList(this.params).then(res => {
          this.loading = 0  // 关闭页面loading占位
          let data = res.data.data
          if (data.list.length === 0) {
            this.noMore = true
          }
          // 完成回调，关闭加载loading占位
          if(done) done(this.noMore)
        })
      }
    }
  }
</script>
