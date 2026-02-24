<template>
	<view :style="colorStyle" class="review-page">
		<view class="page-nav">
			<view class="nav-back" @click="goBack">
				<text class="iconfont icon-xiangzuo"></text>
			</view>
			<view class="nav-title">{{ pageTitle }}</view>
			<view class="nav-space"></view>
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
import { getOrderList, orderComment } from '@/api/order.js';
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
			pageTitle: this.$t(`活动评价`),
			orderUnique: '',
			orderId: '',
			orderLoading: false
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
			this.loadOrderTarget();
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
		loadOrderTarget() {
			if (this.orderLoading || !this.product_id) return;
			this.orderLoading = true;
			this.orderUnique = '';
			this.orderId = '';
			getOrderList({
				type: 3,
				page: 1,
				limit: 50
			})
				.then((res) => {
					const list = Array.isArray(res.data) ? res.data : [];
					const productId = Number(this.product_id);
					for (let i = 0; i < list.length; i += 1) {
						const order = list[i] || {};
						const cartInfo = Array.isArray(order.cartInfo) ? order.cartInfo : [];
						for (let j = 0; j < cartInfo.length; j += 1) {
							const item = cartInfo[j] || {};
							const productInfo = item.productInfo || {};
							const itemProductId = Number(
								productInfo.product_id || productInfo.id || item.product_id || item.productId
							);
							if (itemProductId && itemProductId === productId) {
								this.orderUnique = item.unique || '';
								this.orderId = order.order_id || '';
								break;
							}
						}
						if (this.orderUnique) break;
					}
					this.orderLoading = false;
				})
				.catch(() => {
					this.orderLoading = false;
				});
		},
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
			if (!this.orderUnique) {
				return this.$util.Tips({
					title: this.$t(`暂无可评价订单`)
				});
			}
			const payload = {
				comment: this.content.trim(),
				product_score: 5,
				service_score: 5,
				pics: [],
				unique: this.orderUnique
			};
			uni.showLoading({
				title: this.$t(`正在发布评论`)
			});
			orderComment(payload)
				.then(() => {
					uni.hideLoading();
					this.$util.Tips({
						title: this.$t(`感谢您的评价`),
						icon: 'success'
					});
					this.content = '';
					this.getProductReplyList(true);
					this.loadOrderTarget();
				})
				.catch((err) => {
					uni.hideLoading();
					this.$util.Tips({
						title: err || this.$t(`评论提交失败`)
					});
				});
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
