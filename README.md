# vue-scroller

> 基于vue2.0的上拉加载&下拉刷新组件  

### 使用方法：  

#### template  

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

#### script  
  
    1.  import Scroller from './vue-scroller/index.vue'  

    2.  components: {  
            Scroller  
        }  

    3.  data() {  
            return {  
                loading: true,  // 页面加载的状态  
                noMore: false,  // 没有更多了  
                list: [],       // 页面li数据  
                page: 1         // 初始化页码
            }  
        }  

    4.  methods: {  
        /* 触发刷新时的处理函数 */
        onRefresh (done) {  
            this.page = 1  
            this.noMore = false  
            this.getData('refresh', done)  
        },  
        // 触发加载更多时的处理函数   
        onLoadmore (done) {  
            this.page++  
            this.getData('loadmore', done)  
        },  
        // 获取数据  
        getData (type, done) {  
            // 初始化加载  
            if (!type) this.loading = true  
            // 请求数据 ( 建议使用axios )  
            const params = {  
                page: this.page  
            }  
            axios.get('url', { params: params })  
                .then(res => {  
                    if (!type) this.loading = false  // 初始化加载完成  
                    
                    let data = res.data.data  
                    if (this.page === 1) {  
                        this.list = data.list  
                    } else {  
                        this.list = [].concat(this.list, data.list)  
                    }  

                    // 如没有更多了，需将 this.noMore 置为 true  
                    if (data.list.length === 0) {  
                        this.noMore = true  
                    }  
                    
                    // 完成请求后，手动执行done()，关闭加载占位  
                    if(done) done(this.noMore)  
                }  
            )  
        }  
    }  

    5.  created () {
        this.getData()
    }
    
