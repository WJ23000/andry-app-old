<template lang="pug">
	view.andry-mine
		view.u-search-box
			<!-- #ifdef APP-PLUS || H5 -->
			u-navbar(
				:is-back="false", 
				:title="top>40 ? '我的' : ''", 
				:border-bottom="false",
				:background="headerBg",
				:immersive="true")
				view.u-flex.u-row-right.w100
					u-icon(name="setting", :size="48")
					view.icon-width.u-flex.u-row-center
						u-icon(name="bell", size="48")
						u-badge(
							size="mini", 
							type="error", 
							:count="messageCount", 
							:offset="badgeOffset")
			<!-- #endif -->
			view.u-flex.user-box.u-p-l-30.u-p-r-20.u-p-b-30
				view.avatar.u-m-r-10
					u-avatar(:src="userAvatar", size="140")
				view.u-flex-1
					view.u-font-18.u-p-b-20 {{ userName }}
					view.u-font-14.u-tips-color
						u-icon(name="phone", size="24")
						text {{ phone }}
				view.u-m-l-10.u-p-10
					view.sign 签到领积分
		view.u-m-t-20.order-grid
			u-section(title="我的订单", sub-title="查看全部", :show-line="false", @click="toOrder")
			u-grid(:col="4", :border="false")
				u-grid-item(v-for="(item, index) in orderGridList", :index="index", :key="index", @click="toOrder(item.id)")
					u-icon(:name="item.icon", :size="46")
					text.grid-text {{ item.name }}
		view.u-m-t-20.other-grid
			u-grid(:col="4", :border="false")
				u-grid-item(
					v-for="(item, vindex) in otherGridList", 
					:index="vindex", 
					:key="vindex",
					@click="toNextPage(item.name)")
					u-icon(:name="item.icon", :size="46")
					text.grid-text {{ item.name }}
		u-divider.divider(bg-color="", half-width="280") 猜你喜欢
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
		u-modal(
			v-model="wechatShow",
			width="486",
			:mask-close-able="true",
			:show-title="false",
			:show-confirm-button="false",
			:show-cancel-button="false")
			image.wechat-qrcode(:src="userQrcode")
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import { GOODS_DATA } from "@/model";
@Component({
	components: {
	}
})
export default class Mine extends Vue {
	top = 0;
	headerBg = {
		backgroundColor: ""
	};
	userAvatar = require("@/static/user.png");
	userQrcode = require("@/static/my-qrcode.jpg");
	userName = "zz23000";
	phone = "188xxxxxxxx";
	messageCount = 10;
	badgeOffset = [10, 10];
	show = true;

	orderGridList = [
		{ id: 0, name: "待支付", icon: "integral" },
		{ id: 1, name: "待收货", icon: "kefu-ermai" },
		{ id: 2, name: "待评价", icon: "coupon" },
		{ id: 3, name: "售后/退款", icon: "gift" }
	];
	otherGridList = [
		{ name: "我的会员", icon: "level" },
		{ name: "我的收藏", icon: "star" },
		{ name: "收货地址", icon: "map" },
		{ name: "联系客服", icon: "kefu-ermai" },
		{ name: "礼品卡", icon: "coupon" },
		{ name: "人才招募", icon: "man-add" },
		{ name: "意见反馈", icon: "chat" },
		{ name: "扫一扫", icon: "scan" },
		{ name: "关注公众号", icon: "weixin-fill" }
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
	
	wechatShow = false

	// 监听页面滚动
	onPageScroll(e) {
		this.top = e.scrollTop;
		if(e.scrollTop>40) {
			this.headerBg.backgroundColor = "#ffffff"
		}else {
			this.headerBg.backgroundColor = ""
		}
	}

	// 查看全部订单
	toOrder(id) {
		const current = id ? id : 0;
		uni.navigateTo({
			url: `/pages/Order/Index?current=${current}`
		});
	}

	// 去往其他模块
	toNextPage(name) {
		switch (name) {
			case "我的收藏":
				uni.navigateTo({
					url: "/pages/Collection/Index"
				});
				break;
			case "收货地址":
				uni.navigateTo({
					url: "/pages/Address/Index"
				});
				break;
			case "人才招募":
				uni.navigateTo({
					url: "/pages/TalentRecruit/Index"
				});
				break;
			case "意见反馈":
				uni.navigateTo({
					url: "/pages/FeedBack/Index"
				});
				break;
			case "关注公众号":
				this.wechatShow = true
				break;
			default:
				break;
		}
	}

	created() {
		this.getList();
	}

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
}
</script>

<style lang="scss" scoped>
page {
	background-color: #ededed;
}
.andry-mine {
	.u-search-box {
		background: $andry-bg-image;
		background-size: cover;
	}
	.icon-width {
		position: relative;
		width: 54px;
		height: 44px;
		&:active {
			background-color: #ededed;
		}
	}
	.sign {
		padding: 4rpx 12rpx;
		font-size: 24rpx;
		background: #ffffff;
		border-radius: 50px;
	}
	.user-box {
		padding-top: 120rpx;
		.avatar {
			height: 140rpx;
		}
		.u-flex-1 {
			margin-left: 8rpx;
		}
	}
	.u-tips-color {
		color: #333333
	}
	.order-grid,
	.other-grid {
		background: #ffffff;
		margin: 8rpx 24rpx;
		padding: 28rpx 24rpx 8rpx 24rpx;
		border-radius: 8rpx;
	}
	.grid-text {
		font-size: 24rpx;
		margin-top: 4rpx;
		color: #5d656b;
	}
	.divider {
		padding: 12rpx 0rpx;
	}
	.wrap {
		padding: 0rpx 12rpx;
	}
	.w100 {
		width: 100%;
	}
	.wechat-qrcode {
		margin: 48rpx 0rpx 40rpx 42rpx;
		width: 400rpx;
		height: 400rpx;
	}
}
</style>
