<template>
	<view :style="colorStyle" class="review-page">
		<view class="review-header">
			<view class="review-title">{{ pageTitle }}</view>
		</view>
		<view class="evaluate-list">
			<userEvaluation :reply="reply" :showStar="false"></userEvaluation>
			<view class="loadingicon acea-row row-center-wrapper" v-if="reply.length > 0">
				<text class="loading iconfont icon-jiazai" :hidden="loading == false"></text>{{ loadTitle }}
			</view>
			<view class="noCommodity" v-if="reply.length == 0">
				<view class="emptyBox">
					<image :src="imgHost + '/statics/images/noMessage.png'"></image>
				</view>
				<view class="text">
					{{ $t(`暂无评论`) }}
				</view>
			</view>
		</view>
		<view class="review-form">
			<view class="form-title">{{ $t(`发布评论`) }}</view>
			<textarea
				v-model="content"
				class="form-textarea"
				:placeholder="$t(`写下你的评价`)"
				placeholder-class="placeholder"
			></textarea>
			<button class="submit-btn bg-color" @click="submitReview">{{ $t(`发布评论`) }}</button>
		</view>
	</view>
</template>

<script>
import { getReplyList } from '@/api/store.js';
import userEvaluation from '@/components/userEvaluation';
import colors from '@/mixins/color.js';
import { HTTP_REQUEST_URL } from '@/config/app';
import { toLogin } from '@/libs/login.js';
import { mapGetters } from 'vuex';

export default {
	components: {
		userEvaluation
	},
	mixins: [colors],
	computed: mapGetters(['isLogin']),
	data() {
		return {
			imgHost: HTTP_REQUEST_URL,
			product_id: 0,
			reply: [],
			type: 1,
			loading: false,
			loadend: false,
			loadTitle: this.$t(`加载更多`),
			page: 1,
			limit: 20,
			content: '',
			pageTitle: this.$t(`活动评价`)
		};
	},
	onLoad(options) {
		if (!options.product_id) {
			return this.$util.Tips(
				{
					title: this.$t(`缺少参数`)
				},
				{
					tab: 3,
					url: 1
				}
			);
		}
		this.product_id = options.product_id;
		const name = decodeURIComponent(options.name || '');
		if (name) this.pageTitle = name;
	},
	onShow() {
		if (this.isLogin) {
			this.getProductReplyList(true);
		} else {
			toLogin();
		}
	},
	methods: {
		getProductReplyList(reset) {
			if (this.loadend && !reset) return;
			if (this.loading) return;
			this.loading = true;
			this.loadTitle = '';
			if (reset) {
				this.page = 1;
				this.loadend = false;
				this.reply = [];
			}
			getReplyList(this.product_id, {
				page: this.page,
				limit: this.limit,
				type: this.type
			})
				.then((res) => {
					const list = res.data || [];
					const loadend = list.length < this.limit;
					this.reply = this.$util.SplitArray(list, this.reply);
					this.$set(this, 'reply', this.reply);
					this.loading = false;
					this.loadend = loadend;
					this.loadTitle = loadend ? this.$t(`没有更多内容啦~`) : this.$t(`加载更多`);
					this.page = this.page + 1;
				})
				.catch(() => {
					this.loading = false;
					this.loadTitle = this.$t(`加载更多`);
				});
		},
		submitReview() {
			if (!this.isLogin) return toLogin();
			if (!this.content.trim()) {
				return this.$util.Tips({
					title: this.$t(`请填写评价内容`)
				});
			}
			uni.showToast({
				title: this.$t(`评论提交接口待接入`),
				icon: 'none'
			});
			this.content = '';
		}
	},
	onReachBottom() {
		this.getProductReplyList(false);
	}
};
</script>

<style lang="scss" scoped>
.review-page {
	min-height: 100vh;
	background-color: #fff;
}

.review-header {
	padding: 24rpx 30rpx 12rpx;
}

.review-title {
	font-family: SemiBold;
	font-size: 32rpx;
	color: rgba(17, 59, 46, 0.9);
}

.review-form {
	padding: 24rpx 30rpx 40rpx;
	background-color: #fff;
}

.form-title {
	font-size: 28rpx;
	color: rgba(17, 59, 46, 0.9);
	margin-bottom: 16rpx;
}

.form-textarea {
	width: 100%;
	min-height: 180rpx;
	padding: 18rpx;
	border-radius: 18rpx;
	background: rgba(246, 251, 247, 0.9);
	border: 1rpx solid rgba(17, 59, 46, 0.08);
	box-sizing: border-box;
	font-size: 26rpx;
	color: #303133;
}

.submit-btn {
	margin-top: 20rpx;
	height: 86rpx;
	line-height: 86rpx;
	border-radius: 43rpx;
	color: #fff;
	font-size: 28rpx;
}
</style>
