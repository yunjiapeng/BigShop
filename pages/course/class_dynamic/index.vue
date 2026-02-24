<template>
	<view :style="colorStyle" class="dynamic-page">
		<view class="page-nav">
			<view class="nav-back" @click="goBack">
				<text class="iconfont icon-xiangzuo"></text>
			</view>
			<view class="nav-title">{{ pageTitle }}</view>
			<view class="nav-space"></view>
		</view>
		<view class="dynamic-list" v-if="list.length">
			<navigator
				v-for="(item, idx) in list"
				:key="item.id || idx"
				class="dynamic-card"
				:url="'/pages/extension/news_details/index?id=' + item.id"
				hover-class="none"
			>
				<view class="dynamic-media" v-if="item.image_input && item.image_input.length">
					<image class="dynamic-img" :src="item.image_input[0]" mode="aspectFill"></image>
				</view>
				<view class="dynamic-info">
					<view class="dynamic-text">{{ item.title || $t(`班级动态`) }}</view>
					<view class="dynamic-time" v-if="formatGalleryTime(item)">{{ formatGalleryTime(item) }}</view>
				</view>
			</navigator>
			<view class="loadingicon acea-row row-center-wrapper" v-if="list.length > 0">
				<text class="loading iconfont icon-jiazai" :hidden="loading == false"></text>{{ loadTitle }}
			</view>
		</view>
		<view class="noCommodity" v-else>
			<view class="emptyBox">
				<image :src="imgHost + '/statics/images/noMessage.png'"></image>
			</view>
			<view class="text">{{ $t(`暂无班级动态`) }}</view>
		</view>
	</view>
</template>

<script>
import { getArticleList } from '@/api/api.js';
import colors from '@/mixins/color.js';
import { HTTP_REQUEST_URL } from '@/config/app';
import { toLogin } from '@/libs/login.js';
import { mapGetters } from 'vuex';

export default {
	mixins: [colors],
	computed: mapGetters(['isLogin']),
	data() {
		return {
			imgHost: HTTP_REQUEST_URL,
			pageTitle: this.$t(`班级动态`),
			list: [],
			page: 1,
			limit: 20,
			loading: false,
			loadend: false,
			loadTitle: this.$t(`加载更多`)
		};
	},
	onLoad(options) {
		const name = decodeURIComponent(options.name || '');
		if (name) this.pageTitle = name;
	},
	onShow() {
		if (this.isLogin) {
			this.loadList(true);
		} else {
			toLogin();
		}
	},
	methods: {
		goBack() {
			uni.navigateBack({
				delta: 1
			});
		},
		loadList(reset) {
			if (this.loading) return;
			if (this.loadend && !reset) return;
			this.loading = true;
			this.loadTitle = '';
			if (reset) {
				this.page = 1;
				this.loadend = false;
				this.list = [];
			}
			getArticleList(0, {
				page: this.page,
				limit: this.limit
			})
				.then((res) => {
					const list = Array.isArray(res.data) ? res.data : [];
					const filtered = this.filterRecent(list);
					this.list = this.list.concat(filtered);
					const loadend = list.length < this.limit || this.reachOldLimit(list);
					this.loadend = loadend;
					this.loadTitle = loadend ? this.$t(`没有更多内容啦~`) : this.$t(`加载更多`);
					this.page = this.page + 1;
					this.loading = false;
				})
				.catch(() => {
					this.loading = false;
					this.loadTitle = this.$t(`加载更多`);
				});
		},
		getItemTime(item) {
			if (!item) return 0;
			const raw = item.add_time || item.upload_time || item.created_at;
			if (!raw) return 0;
			const num = Number(raw);
			if (Number.isNaN(num)) {
				const date = new Date(raw);
				return Number.isNaN(date.getTime()) ? 0 : date.getTime();
			}
			return num > 1000000000000 ? num : num * 1000;
		},
		filterRecent(list) {
			const cutoff = Date.now() - 30 * 24 * 60 * 60 * 1000;
			return list.filter((item) => {
				const time = this.getItemTime(item);
				return time && time >= cutoff;
			});
		},
		reachOldLimit(list) {
			const cutoff = Date.now() - 30 * 24 * 60 * 60 * 1000;
			return list.some((item) => {
				const time = this.getItemTime(item);
				return time && time < cutoff;
			});
		},
		formatGalleryTime(item) {
			const time = this.getItemTime(item);
			if (!time) return '';
			const date = new Date(time);
			const y = date.getFullYear();
			const m = `${date.getMonth() + 1}`.padStart(2, '0');
			const d = `${date.getDate()}`.padStart(2, '0');
			return `${y}-${m}-${d}`;
		},
	},
	onReachBottom() {
		this.loadList(false);
	}
};
</script>

<style lang="scss" scoped>
.dynamic-page {
	min-height: 100vh;
	background-color: #f6fbf7;
}

.page-nav {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 24rpx 30rpx 16rpx;
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

.dynamic-list {
	padding: 0 24rpx 30rpx;
	display: flex;
	flex-direction: column;
	gap: 18rpx;
}

.dynamic-card {
	border-radius: 22rpx;
	overflow: hidden;
	background: rgba(255, 255, 255, 0.92);
	border: 1rpx solid rgba(17, 59, 46, 0.08);
}

.dynamic-media {
	width: 100%;
}

.dynamic-img {
	width: 100%;
	height: 320rpx;
}

.dynamic-info {
	padding: 16rpx 18rpx 18rpx;
}

.dynamic-text {
	font-size: 26rpx;
	color: rgba(17, 59, 46, 0.88);
	line-height: 36rpx;
}

.dynamic-time {
	margin-top: 10rpx;
	font-size: 22rpx;
	color: rgba(17, 59, 46, 0.55);
}
</style>
