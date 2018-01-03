# vue-refresh-loadmore

> 基于vue2.0的上拉加载&下拉刷新组件  
  
  测试阶段，请勿下载  
  
**注意：需要给此组件的父级正确的height**  
**注意：需要给此组件的父级正确的height**  
**注意：需要给此组件的父级正确的height**  
    
    
    
## 使用方法：  

#### 1、install  
    $ npm install vue-refresh-loadmore --save
    
#### 2、template  

    <scroller  
        :loading="loading"  
        :noMore="noMore"  
        @onRefresh="onRefresh"  
        @onLoadmore="onLoadmore">  
        <ul>  
            <li v-for="(item, index) in list" :key="index">...</li>  
        </ul>  
    </scroller>  
  
#### 3、script  
 
    import Scroller from 'vue-refresh-loadmore'  
    export default {
        components: { Scroller },  
        data () {  
            return {  
                loading: true,  // 页面加载的状态  
                noMore: false,  // 没有更多了  
                list: [],       // 页面li数据  
                page: 1         // 初始化页码
            }  
        },  
        methods: {  
            onRefresh (done) {  // 触发刷新时的处理函数
                this.page = 1  
                this.noMore = false  
                this.getData('refresh', done)  
            },  
            onLoadmore (done) {  // 触发加载更多时的处理函数
                this.page++  
                this.getData('loadmore', done)  
            },  
            getData (type, done) {  
                if (!type) this.loading = true    // 初始化加载  
                
                // 请求数据 ( 建议使用axios )  
                const params = {  
                    page: this.page  
                }  
                axios.get('url', { params: params }).then(res => {  
                    if (!type) this.loading = false  // 初始化完成  
                    
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
                })  
            }  
        },  
        created () {
            this.getData()
        }
    }
