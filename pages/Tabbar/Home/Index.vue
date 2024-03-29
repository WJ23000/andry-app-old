<template lang="pug">
	view.andry-home
		view.search-sticky(v-if="top > 280")
			u-search(
				placeholder="护肤品", 
				v-model="keyword", 
				:show-action="false",
				:disabled="true",
				@click="onSearch")
		view.header
			view.search
				u-search(
					placeholder="护肤品", 
					v-model="keyword", 
					:show-action="false",
					bg-color="#ffffff",
					:disabled="true",
					@click="onSearch")
			view.swiper
				u-swiper(:list="bannerList")
			view.grid
				Grid(:gridList="gridList")
		view.tabs
			u-sticky(:offset-top="stickyTop", @fixed="onFixed", @unfixed="onUnfixed")
				u-tabs(
					:list="tabList",
					:is-scroll="false",
					:current="tabCurrent",
					:show-bar="true",
					bar-width="100",
					inactive-color="#5d656b",
					active-color="#ff0000",
					:bg-color="isFixed ? '#ffffff' : '#ededed'",
					@change="tabChange")
		view.wrap
			goods-list(
				:leftHeight="leftHeight",
				:rightHeight="rightHeight",
				:leftList="leftList",
				:rightList="rightList",
				:loadTxt="ajax.loadTxt",
				@onHeight="onHeight",
				@onClick="onClick")
		back-top(:top="top")
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import Grid from "./components/Grid.vue";
import { BANNER_DATA, GRID_DATA, GOODS_DATA } from "@/model";
@Component({
	components: {
		Grid
	}
})
export default class Home extends Vue {
	top = 0;
	keyword = "";
	bannerList = BANNER_DATA;
	gridList = GRID_DATA;
	isFixed = false; // 是否吸顶
	tabCurrent = 0;
	tabList = [
		{
			name: "猜你喜欢"
		},
		{
			name: "精选新品"
		},
		{
			name: "特价促销"
		},
		{
			name: "清凉一夏"
		}
	];

	// 商品推荐
	leftHeight = 0;
	rightHeight = 0;
	leftList: any[] = [];
	rightList: any[] = [];
	ajax = {
		// 是否可以加载
		load: true,
		// 加载中提示文字
		loadTxt: "",
		// 每页的请求条件
		rows: 10,
		// 页码
		page: 1
	};

	get stickyTop() {
		return uni.getSystemInfoSync().screenWidth == 768
			? 44
			: uni.getSystemInfoSync().screenWidth == 320
			? -17
			: uni.getSystemInfoSync().screenWidth == 360
			? -6
			: 0;
	}

	onLoad() {
		this.getList();
	}

	// 监听页面滚动
	onPageScroll(e) {
		this.top = e.scrollTop;
	}

	// tab切换
	tabChange(index) {
		this.tabCurrent = index;
	}

	// 监听吸顶/取消吸顶
	onFixed() {
		this.isFixed = true;
	}
	onUnfixed() {
		this.isFixed = false;
	}

	// 商品推荐列表
	// 触底触发
	onReachBottom() {
		this.getList();
	}
	// 下拉刷新
	onPullDownRefresh() {
		// 正常情况下接口返回应该很会很快。故意延迟调用，让用户有在刷新的体验感
		setTimeout(() => {
			this.ajax.page = 1;
			this.leftHeight = 0;
			this.rightHeight = 0;
			this.ajax.load = true;
			this.getList();
		}, 800);
	}
	// 监听高度变化
	onHeight(height, tag) {
		if (tag == "left") {
			this.leftHeight += height;
		} else {
			this.rightHeight += height;
		}
	}
	// 组件点击事件
	onClick(index, tag) {
		console.log(index, tag);
		// 对应的数据
		if (tag == "left") {
			console.log(this.leftList[index]);
		} else {
			console.log(this.rightList[index]);
		}
		let direction = {
			left: "左",
			right: "右"
		};
		uni.showToast({
			title: `${direction[tag]}侧列表第${index + 1}个被点击`,
			icon: "none"
		});
	}
	// 获取数据
	getList() {
		/*
			无真实数据，当前由 setTimeout 模拟异步请求、
			自行替换 请求方法将数据 传入 addList() 方法中
			自行解决数据格式，自行修改组件内布局和内容绑定
		*/
		if (!this.ajax.load) {
			return;
		}
		this.ajax.load = false;
		this.ajax.loadTxt = "加载中";

		setTimeout(() => {
			// 生成随机数方法
			let random = (min = 0, max) => {
				return Math.floor(Math.random() * max) + min;
			};
			
			let res: any = [];
			GOODS_DATA.forEach(item=>{
				const price = random(9, 9999) + Math.random();
				item.money = price.toFixed(2)
			})
			this.addList(GOODS_DATA);
		}, 300);
	}
	addList(res) {
		// 获取到的数据，请注意数据结构
		if (!res || res.length < 1) {
			this.ajax.loadTxt = "没有更多了";
			return;
		}

		// 左右列表高度的差
		let differ = this.leftHeight - this.rightHeight;
		// 计算差值，用于确定优先插入那一边
		let differVal = 0;

		// 初始化左右列表的数据
		let i = differ > 0 ? 1 : 0;

		let [left, right]: any = [[], []];

		// 获取插入的方向
		let getDirection = index => {
			/* 左侧高度大于右侧超过 600px 时，则前3条数据都插入到右边 */
			if (differ >= 600 && index < 3) {
				differVal = 1;
				return "right";
			}

			/* 右侧高度大于左侧超过 600px 时，则前3条数据都插入到左边 */
			if (differ <= -600 && index < 3) {
				differVal = -1;
				return "left";
			}

			/* 左侧高度大于右侧超过 350px 时，则前2条数据都插入到右边 */
			if (differ >= 350 && index < 2) {
				return "right";
			}
			/* 右侧高度大于左侧超过 350px 时，则前2条数据都插入到左边 */
			if (differ <= -350 && index < 2) {
				differVal = -1;
				return "left";
			}

			/* 当前数据序号为偶数时，则插入到左边 */
			if ((i + differVal) % 2 == 0) {
				return "left";
			} else {
				/* 当前数据序号为奇数时，则插入到右边 */
				return "right";
			}
		};

		// 将数据源分为左右两个列表，容错差值请自行根据项目中的数据情况调节
		res.forEach((item, index) => {
			if (getDirection(index) == "left") {
				//console.log(`差值：${differ},方向：left，序号${index}`)
				left.push(item);
			} else {
				//console.log(`差值：${differ},方向：right，序号${index}`)
				right.push(item);
			}
			i++;
		});

		// 将左右列表的数据插入，第一页时则覆盖
		if (this.ajax.page == 1) {
			this.leftList = left;
			this.rightList = right;
			uni.stopPullDownRefresh();
		} else {
			this.leftList = [...this.leftList, ...left];
			this.rightList = [...this.rightList, ...right];
		}

		this.ajax.load = true;
		this.ajax.loadTxt = "上拉加载更多";
		this.ajax.page++;
	}
	
	// 搜索
	onSearch() {
		uni.navigateTo({
			url: "/pages/Search/Index"
		});
	}
}
</script>

<style lang="scss">
page {
	background: #ededed;
}
</style>
<style lang="scss" scoped>
.andry-home {
	.header {
		background: linear-gradient(bottom, #ededed, #ff6666, #ff0000);
		padding: 0rpx 24rpx;
		.search {
			height: 88rpx;
			line-height: 88rpx;
		}
		.grid {
			margin-top: 12rpx;
			padding: 12rpx 0px;
			background: #ffffff;
			border-radius: 8rpx;
		}
	}
	.search-sticky {
		position: fixed;
		top: 0;
		width: 100%;
		height: 88rpx;
		line-height: 88rpx;
		background: #ffffff;
		padding: 0rpx 24rpx;
		z-index: 1000;
	}
	.wrap {
		padding: 0rpx 12rpx;
		background: #ededed;
		.waterfall-box {
			padding: 20rpx 0rpx;
			box-sizing: border-box;

			> view {
				padding: 0 10rpx;
			}
		}
		.h-flex-x {
			display: flex;
			flex-direction: row;
			flex-wrap: nowrap;
			justify-content: flex-start;
			align-items: flex-start;
			align-content: flex-start;
			&.h-flex-2 {
				> view {
					width: 50%;
				}
			}
		}
		.load-txt {
			padding: 0 0 20rpx 0;
			height: 100%;
			text-align: center;
			color: #999;
			font-size: 24rpx;
		}
	}
}
</style>
