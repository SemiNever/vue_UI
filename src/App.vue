<template>
  <div id="app">
  <view-box>
    <x-header class='header' slot='header' :left-options="{showBack:false}">
      <div slot='left'>直播</div>
      <div>网易</div>
      <div slot='right'>搜索</div>
    </x-header>
    <sc :lock-y="true">
    <div>
    <tab class='tab'>
      <tab-item selected>推荐</tab-item>
      <tab-item>要闻</tab-item>
      <tab-item>军事</tab-item>
      <tab-item>游戏</tab-item>
      <tab-item>科技</tab-item>
      <tab-item>网络</tab-item>

    </tab>
    </div>
    </sc>
       <scroller class='my-scroller' 
       :on-refresh='refresh'
        refresh-text='loading'
        :on-infinite='infnite'
        ref="myRef"
       >
         <swiper :list='swiperList' v-model='swiperIndex' :loop='true'></swiper>
    <marquee class='my-marquee'>
      <marquee-item v-for="list in marqueeList">{{list.title}}</marquee-item>
    </marquee>
    <panel :list='dataList'> 
    </panel>
    <panel :list='moreDataList'> 
    </panel>
       </scroller>
    
    <tabbar slot='bottom'>
      <tabbar-item>
        <img src="./assets/首页.png" slot='icon'>
        <span slot='label'>首页</span>
      </tabbar-item>
         <tabbar-item>
        <img src="./assets/推荐.png" slot='icon'>
        <span slot='label'>推荐</span>
      </tabbar-item>
         <tabbar-item>
        <img src="./assets/我的.png" height="200" width="200" slot='icon'>
        <span slot='label'>我的</span>
      </tabbar-item>
    </tabbar>
  </view-box>
  <router-view></router-view>
  </div>
</template>

<script>
import {ViewBox,Marquee,MarqueeItem,XHeader,Tabbar,TabbarItem,Tab,TabItem,Scroller as sc,Swiper,Panel} from 'vux';
var refreshKey=['A','B01','B02','B03','B04','B05','B06','B07','B08','B09','B010'];
var key=0;
var start=0;
var end=start+9;
var keyValue='A';

var initLoaded=false;//初始化数据是否加载完成
function getRefreshKey(){
 key++; 
 if(key==refreshKey.length){
    key=0;
  }
keyValue =refreshKey[key];
}


export default {
  name: 'app',
  components:{
  	ViewBox,
  	XHeader,
    Tabbar,
    TabbarItem,
    Tab,
    TabItem,
    sc,
    Swiper,
    Marquee,
    MarqueeItem,
    Panel
  },
  created(){
    this.$jsonp("http://3g.163.com/touch/jsonp/sy/recommend/0-9.html").then(data=>{
      //轮播图数据
      this.swiperList=data.focus.filter(
            item=>{
          return item.addata=== null&&item.picInfo[0];
            }).map(item=>{
              return {
             url: item.link,
             img: item.picInfo[0].url,
             title:item.title
              }
            });
     // 首页新闻列表数据
     this.dataList=data.list.filter(
            item=>{
          return item.addata=== null&&item.picInfo[0];
            }).map(item=>{
              return {
             url: item.link,
             src: item.picInfo[0].url,
             title:item.title,
             desc:item.digest
              }
            });
            initLoaded=true;

   //滚动条数据
   this.marqueeList=data.live.filter(
            item=>{
          return item.addata=== null;
            }).map(item=>{
              return {
             title:item.title
              }
            });
    });
  },
  data(){
    var swiperList=[];
    var marqueeList=[];
    var dataList=[];
    var moreDataList=[];
    return{
      swiperList:[],
      swiperIndex:0,
      dataList:dataList,
      marqueeList:marqueeList,
      moreDataList:moreDataList
    }
  },
  methods:{
    refresh(){
      getRefreshKey();
      this.$jsonp('http://3g.163.com/touch/jsonp/sy/recommend/0-9.html',{
        miss:'00',
        refresh:keyValue
      }).then(data=>{
        // console.log(this.$refs.myRef);
         this.dataList=data.list.filter(
            item=>{
          return item.addata=== null&&item.picInfo[0];
            }).map(item=>{
              return {
             url: item.link,
             src: item.picInfo[0].url,
             title:item.title,
             desc:item.digest
              }
            });
            this.$refs.myRef.finishPullToRefresh();
            this.$vux.toast.show(`已更新${this.dataList.length}条数据`);
      })
    },
    infnite(){
      if(!initLoaded){
        this.$refs.myRef.finishInfinite();
        return;
      }
      //上拉加载更多数据
      setTimeout(()=>{
        this.$jsonp(`http://3g.163.com/touch/jsonp/sy/recommend/${start}-${end}.html`,{
        miss:'00',
        refresh:keyValue
      }).then(data=>{
this.moreDataList=data.list.filter(
            item=>{
          return item.addata=== null&&item.picInfo[0];
            }).map(item=>{
              return {
             url: item.link,
             src: item.picInfo[0].url,
             title:item.title,
             desc:item.digest
              }
            });
    start+=10;
    end=start+9;
    // this.moreDataList=this.moreDataList.concat(this.dataList);
    this.$refs.myRef.finishInfinite();
      });
    }, 500);
      

    }
  }
}
</script>

<style lang="less">
@import '~vux/src/styles/reset.less';
html,body{
	margin:0;
	width:100%;
	height: 100%;
	overflow:hidden;
}
#app{
	height:100%;
  .tab{
    width:200%;
  }
  .weui-media-box__thumb{
    width:60px;
    height:60px;
  }
  .my-marquee{
    margin:10px;
  }
  .my-scroller{
    top:90px;
  }
}

</style>
