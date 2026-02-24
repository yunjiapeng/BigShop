<template>
	<view class="contact-page">
		<view class="page-nav">
			<view class="nav-back" @click="goBack">
				<text class="iconfont icon-xiangzuo"></text>
			</view>
			<view class="nav-title">{{ $t('联系老师') }}</view>
			<view class="nav-space"></view>
		</view>
		<view class="contact-card">
			<image class="contact-qr" :src="contactPng" mode="aspectFill"></image>
			<view class="contact-tip">{{ $t('长按二维码以联系老师') }}</view>
		</view>
	</view>
</template>

<script>
import contactPng from '@/logo/contact.png';

export default {
	data() {
		return {
			contactPng
		};
	},
	methods: {
		goBack() {
			uni.navigateBack({
				delta: 1
			});
		}
	}
};
</script>

<style lang="scss" scoped>
.contact-page {
	min-height: 100vh;
	display: flex;
	flex-direction: column;
	align-items: center;
	background: #f6fbf7;
	padding: 24rpx 24rpx 40rpx;
	box-sizing: border-box;
}

.page-nav {
	width: 100%;
	display: flex;
	align-items: center;
	justify-content: space-between;
	margin-bottom: 24rpx;
}

.nav-back {
	width: 56rpx;
	height: 56rpx;
	border-radius: 28rpx;
	background: rgba(255, 255, 255, 0.9);
	border: 1rpx solid rgba(17, 59, 46, 0.12);
	display: flex;
	align-items: center;
	justify-content: center;
	color: rgba(17, 59, 46, 0.9);
}

.nav-title {
	font-family: SemiBold;
	font-size: 32rpx;
	color: rgba(17, 59, 46, 0.9);
}

.nav-space {
	width: 56rpx;
}

.contact-card {
	width: 100%;
	max-width: 520rpx;
	background: #ffffff;
	border-radius: 24rpx;
	padding: 36rpx 24rpx 32rpx;
	display: flex;
	flex-direction: column;
	align-items: center;
	gap: 18rpx;
	border: 1rpx solid rgba(17, 59, 46, 0.08);
}

.contact-qr {
	width: 320rpx;
	height: 320rpx;
	border-radius: 18rpx;
}

.contact-tip {
	font-size: 24rpx;
	color: rgba(17, 59, 46, 0.78);
	text-align: center;
	line-height: 32rpx;
}
</style>
