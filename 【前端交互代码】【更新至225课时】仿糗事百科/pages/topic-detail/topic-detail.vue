<template>
	<view>
		<!-- 话题介绍 -->
		<topic-info :item="topicInfo"></topic-info>
		<!-- tabbar切换 -->
		<swiper-tab-head 
		:tabBars="tabBars" 
		:tabIndex="tabIndex"
		@tabtap="tabtap"
		scrollItemStyle="width:50%;"
		scrollStyle="border-bottom:0;">
		</swiper-tab-head>
		<!-- 列表 -->
		<view class="topic-detail-list">
			<block v-for="(item,index) in tablist" :key="index">
				<template v-if="tabIndex==index">
					<!-- 列表 -->
					<block v-for="(list,listindex) in item.list" :key="listindex">
						<common-list :item="list" :index="listindex"></common-list>
					</block>
					<!-- 上拉加载 -->
					<load-more :loadtext="item.loadtext"></load-more>
				</template>
			</block>
		</view>
		
	</view>
</template>

<script>
	import topicInfo from "../../components/topic/topic-info.vue";
	import swiperTabHead from "../../components/index/swiper-tab-head.vue";
	import commonList from "../../components/common/common-list.vue";
	import loadMore from "../../components/common/load-more.vue";
	export default {
		components:{
			topicInfo,
			swiperTabHead,
			commonList,
			loadMore
		},
		data() {
			return {
				topicInfo:{
					titlepic:"../../static/demo/topicpic/13.jpeg",
					title:"忆往事，敬余生",
					desc:"我是描述",
					totalnum:1000,
					todaynum:1000,
				},
				tabIndex:0,
				tabBars:[
					{ name:"默认",id:"moren" },
					{ name:"最新",id:"zuixin" },
				],
				tablist:[
					{ loadtext:"上拉加载更多",list:[],firstload:false,page:1},
					{ loadtext:"上拉加载更多",list:[],firstload:false,page:1},
				]
			};
		},
		// 上拉触底事件
		onReachBottom() {
			// 上拉加载
			this.loadmore();
		},
		// 监听下拉刷新 
		onPullDownRefresh(){
			this.getdata();
		},
		onLoad(e) {
			this.__init(JSON.parse(e.detail));
		},
		methods:{
			// 初始化
			__init(obj){
				// 修改标题
				uni.setNavigationBarTitle({ title: obj.title });
				this.topicInfo = obj;
				// 获取列表数据
				this.getList();
			},
			// 获取数据
			async getList(){
				let url = `/topic/${this.topicInfo.id}/post/${this.tablist[this.tabIndex].page}`;
				let [err,res] = await this.$http.get(url,{},{token:true});
				// 错误处理
				if (!this.$http.errorCheck(err,res)) {
					return this.tablist[this.tabIndex].loadtext="上拉加载更多";
				}
				let arr = [];
				let list = res.data.data.list;
				for (let i = 0; i < list.length; i++) {
					arr.push(this.__format(list[i]));
				}
				this.tablist[this.tabIndex].list = this.tablist[this.tabIndex].page > 1 ? this.tablist[this.tabIndex].list.concat(arr) : arr;
				this.tablist[this.tabIndex].firstload = true;
				return	this.tablist[this.tabIndex].loadtext=(list.length < 10) ?"没有更多数据了" : "上拉加载更多";
			},
			// 转换格式
			__format(item){
				return {
					userid:item.user.id,
					userpic:item.user.userpic,
					username:item.user.username,
					isguanzhu:!!item.user.fens.length,
					id:item.id,
					title:item.title,
					type:"img", // img:图文,video:视频
					titlepic:item.titlepic,
					video:false,
					path:item.path,
					share:!!item.share,
					infonum:{
						index:(item.support.length>0) ? (item.support[0].type+1) : 0,
						dingnum:item.ding_count,
						cainum:item.cai_count,
					},
					goodnum:item.ding_count,
					commentnum:item.comment_count,
					sharenum:item.sharenum,
				}
			},
			// 上拉加载
			loadmore(){
				if(this.tablist[this.tabIndex].loadtext!="上拉加载更多") return;
				// 修改状态
				this.tablist[this.tabIndex].loadtext="加载中...";
				// 页数+1
				this.tablist[this.tabIndex].page++;
				// 获取数据
				this.getList();
			},
			// tabbar点击事件
			tabtap(index){
				this.tabIndex=index;
				this.tablist[this.tabIndex].page = 1;
				this.getList();
			},
		}
	}
</script>

<style>

</style>
