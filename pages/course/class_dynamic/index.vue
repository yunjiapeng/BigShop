<template>
	<view :style="colorStyle" class="dynamic-page">
		<view class="dynamic-header">
			<view class="dynamic-title">{{ pageTitle }}</view>
		</view>
		<view class="dynamic-list" v-if="list.length">
			<view class="dynamic-card" v-for="(item, idx) in list" :key="item.id || idx" @click="previewImage(idx)">
				<image class="dynamic-img" :src="item.pic_url" mode="aspectFill"></image>
				<view class="dynamic-info">
					<view class="dynamic-text">{{ item.content || item.desc || item.title || $t(`班级动态`) }}</view>
					<view class="dynamic-time" v-if="formatGalleryTime(item)">{{ formatGalleryTime(item) }}</view>
				</view>
			</view>
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
import { getChildGallery } from '@/api/user.js';
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
			classId: '',
			childUid: '',
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
		this.classId = options.class_id || '';
		this.childUid = options.child_uid || '';
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
			if (!this.childUid) {
				this.loading = false;
				this.loadend = true;
				this.loadTitle = this.$t(`没有更多内容啦~`);
				return;
			}
			getChildGallery({
				child_uid: this.childUid,
				class_id: this.classId,
				page: this.page,
				limit: this.limit
			})
				.then((res) => {
					const data = res && res.data ? res.data : {};
					const list = Array.isArray(data.list) ? data.list : Array.isArray(data) ? data : [];
					this.list = this.list.concat(list);
					const loadend = list.length < this.limit;
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
		formatGalleryTime(item) {
			if (!item) return '';
			const raw = item.upload_time || item.created_at;
			if (!raw) return '';
			const num = Number(raw);
			let date;
			if (Number.isNaN(num)) {
				date = new Date(raw);
			} else {
				date = new Date(num > 1000000000000 ? num : num * 1000);
			}
			if (Number.isNaN(date.getTime())) return '';
			const y = date.getFullYear();
			const m = `${date.getMonth() + 1}`.padStart(2, '0');
			const d = `${date.getDate()}`.padStart(2, '0');
			return `${y}-${m}-${d}`;
		},
		previewImage(idx) {
			const urls = this.list.map((item) => item.pic_url).filter(Boolean);
			if (!urls.length) return;
			uni.previewImage({
				current: urls[idx] || urls[0],
				urls
			});
		}
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

.dynamic-header {
	padding: 24rpx 30rpx 8rpx;
}

.dynamic-title {
	font-family: SemiBold;
	font-size: 32rpx;
	color: rgba(17, 59, 46, 0.9);
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
