<template>
  <view>
    <view class="goods-list">
      <view v-for="(goods, i) in goodsList" :key="i" @click="gotoDetail(goods)">
        <!-- 为 my-goods 组件动态绑定 goods 属性的值 -->
        <my-goods :goods="goods"></my-goods>
      </view>
    </view>
  </view>
</template>

<script>
export default {
  data() {
    return {
      goodsList: [], // 商品列表的数据
      total: 0, // 总数量，用来实现分页
      queryObj: {
        // 请求参数对象
        query: '', // 查询关键词
        cid: '', // 商品分类Id
        pagenum: 1, // 页码值
        pagesize: 10 // 每页显示多少条数据
      },
      // 是否正在请求数据 
      isloading: false
    };
  },
  onLoad(options) {
    // console.log(options)
    // 将页面参数转存到 this.queryObj 对象中
    this.queryObj.query = options.query || '';
    this.queryObj.cid = options.cid || '';
    // console.log(this.queryObj);
    // 调用获取商品列表数据的方法
    this.getGoodsList();
  },
  methods: {
    // 获取商品列表数据的方法
    async getGoodsList(cb) {
      this.isloading=true
      // 发起请求
      const { data: res } = await uni.$http.get('/api/public/v1/goods/search', this.queryObj);
      this.isloading=false;
      // 只要数据请求完毕，就立即按需调用 cb 回调函数
      cb&&cb()
      // console.log(res);
      if (res.meta.status !== 200) return uni.$showMsg();
      // console.log(this.queryObj);
      // 为数据赋值
      this.goodsList = [...this.goodsList,...res.message.goods];
      this.total = res.message.total;
    },
    gotoDetail(goods){
      uni.navigateTo({ url: '/subpkg/goods_detail/goods_detail?goods_id=' + goods.goods_id })
    }
  },
  // 下拉刷新的事件
  onReachBottom() {
    // 判断是否还有下一页数据 
    if (this.queryObj.pagenum * this.queryObj.pagesize >= this.total) return uni.$showMsg('数据加载完毕！')
    if(this.isloading) return
    this.queryObj.pagenum += 1
    this.getGoodsList()
  },
  onPullDownRefresh() {
    // 1. 重置关键数据 
    this.queryObj.pagenum = 1 
    this.total = 0 
    this.isloading = false 
    this.goodsList = [] 
    // 2. 重新发起请求 
    this.getGoodsList(() => uni.stopPullDownRefresh())
  }
};
</script>

<style lang="scss"></style>

