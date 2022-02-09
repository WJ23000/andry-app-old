<template lang="pug">
	view.andry-classify
		<!-- #ifdef APP-PLUS || H5 -->
		u-navbar.u-search-box(
			:is-back="false", 
			title="购物车", 
			:border-bottom="false",
			:background="headerBg",
			:immersive="true")
			template(slot="right")
				view.u-flex.u-row-right.w100
					button.del-btn(:disabled="del") 删除
		<!-- #endif -->
		u-navbar.u-search-box(
			back-text="购物车", 
			:border-bottom="false",
			:background="headerBg",
			:immersive="true")
			template(slot="right")
				view.u-flex.u-row-right.w100
					button.del-btn(:disabled="del") 删除
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
@Component({
	components: {}
})
export default class cart extends Vue {
	headerBg = {
		backgroundColor: "#ffffff"
	};
	
	del = true
}
</script>

<style lang="scss" scoped>
.andry-classify {
	.del-btn {
		background: #ffffff;
		font-size: 28rpx;
		border: 0 !important;
		&::after {
			border: 0 !important;
		}
	}
}
</style>
