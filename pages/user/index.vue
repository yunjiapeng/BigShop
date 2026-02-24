<template>
	<view class="new-users copy-data" :style="{ height: pageHeight }">
		<view class="top">
			<!-- #ifdef MP || APP-PLUS -->
			<view class="sys-head">
				<view class="sys-bar" :style="{ height: sysHeight }"></view>
				<!-- #ifdef MP -->
				<view class="sys-title">{{ $t('个人中心') }}</view>
				<!-- #endif -->
				<view class="bg"></view>
			</view>
			<!-- #endif -->
		</view>
		<view class="mid" style="flex: 1; overflow: hidden">
			<scroll-view scroll-y="true" style="height: 100%">
				<view class="head">
					<view class="user-card">
						<view class="user-info">
							<view>
								<!-- 注释这个是加的bnt -->
								<!-- #ifdef H5 -->
								<!-- <button class="bntImg" v-if="userInfo.is_complete == 0 && isWeixin"
									@click="getWechatuserinfo">
									<image class="avatar" src='/static/images/f.png'></image>
									<view class="avatarName">{{$t('获取头像')}}</view>
								</button> -->
								<!-- #endif -->
								<!-- #ifndef APP-PLUS -->
								<view class="avatar-box" :class="{ on: userInfo.is_money_level }">
									<image class="avatar" :src="userInfo.avatar" v-if="userInfo.avatar" @click="goEdit()"></image>
									<image v-else class="avatar" src="/static/images/f.png" mode="" @click="goEdit()"></image>
									<view class="headwear" v-if="userInfo.is_money_level && userInfo.svip_open">
										<image src="/static/images/headwear.png"></image>
									</view>
								</view>
								<!-- #endif -->
								<!-- #ifdef APP-PLUS -->
								<view class="avatar-box" :class="{ on: userInfo.is_money_level }">
									<image class="avatar" :src="userInfo.avatar" v-if="userInfo.avatar" @click="goEdit()"></image>
									<image v-else class="avatar" src="/static/images/f.png" mode="" @click="goEdit()"></image>
								</view>
								<!-- #endif -->
							</view>
							<view class="info">
								<!-- #ifdef MP || APP-PLUS -->
								<view class="name" v-if="!userInfo.uid" @click="openAuto" style="height: 100%; display: flex; align-items: center">
									{{ $t('请点击授权') }}
								</view>
								<!-- #endif -->
								<!-- #ifdef H5 -->
								<view class="name" v-if="!userInfo.uid" @click="openAuto" style="height: 100%; display: flex; align-items: center">
									{{ $t(isWeixin ? '请点击授权' : '请点击登录') }}
								</view>
								<!-- #endif -->
								<view class="name" v-if="userInfo.uid">
									<text class="line1 nickname">{{ userInfo.nickname }}</text>
								</view>
								<view class="num" v-if="userInfo.phone" @click="goEdit()">
									<view class="num-txt">{{ userInfo.phone }}</view>
								</view>
								<!-- #ifdef MP -->
								<button class="phone" v-if="!userInfo.phone && isLogin" open-type="getPhoneNumber" @getphonenumber="getphonenumber">{{ $t(`绑定手机号`) }}</button>
								<!-- #endif -->
								<!-- #ifndef MP -->
								<view class="phone" v-if="!userInfo.phone && isLogin" @tap="bindPhone">
									{{ $t('绑定手机号') }}
								</view>
								<!-- #endif -->
							</view>
							<view class="message" v-if="isLogin">
								<navigator url="/pages/users/user_info/index" hover-class="none">
									<view class="iconfont icon-shezhi"></view>
								</navigator>
							</view>
						</view>
					</view>
					<view class="profile-list">
						<view class="profile-item" @click="toggleOrderMenu">
							<view class="profile-text">{{ $t('订单中心') }}</view>
							<view class="profile-right" @click.stop>
								<view class="profile-tip" @click="goMenuPage('/pages/goods/order_list/index')">{{ $t('查看全部') }}</view>
								<text class="iconfont icon-jiantou"></text>
							</view>
						</view>
						<view class="order-submenu" v-if="showOrderMenu">
							<view class="order-submenu-grid">
								<view class="order-submenu-item" v-for="(item, index) in orderMenu" :key="index" @click="goMenuPage(item.url)">
									<view class="order-submenu-icon">
										<text class="iconfont" :class="item.img"></text>
										<text class="order-status-num" v-if="item.num > 0">{{ item.num }}</text>
									</view>
									<view class="order-submenu-text">{{ $t(item.title) }}</view>
								</view>
							</view>
						</view>
						<view class="profile-item" @click="goMenuPage('/pages/users/user_coupon/index')">
							<view class="profile-text">{{ $t('优惠券') }}</view>
							<text class="iconfont icon-jiantou"></text>
						</view>
						<view class="profile-item" @click="goMenuPage('/pages/users/user_integral/index')">
							<view class="profile-text">{{ $t('积分') }}</view>
							<text class="iconfont icon-jiantou"></text>
						</view>
						<view class="profile-item" @click="goMenuPage('/pages/goods/order_list/index?status=1&verify=1')">
							<view class="profile-text">{{ $t('核销') }}</view>
							<text class="iconfont icon-jiantou"></text>
						</view>
						<view class="profile-item" @click="contactTeacher">
							<view class="profile-text">{{ $t('联系老师') }}</view>
							<text class="iconfont icon-jiantou"></text>
						</view>
					</view>
				</view>
				<view class="uni-p-b-98"></view>
			</scroll-view>
			<editUserModal :isShow="editModal" @closeEdit="closeEdit" @editSuccess="editSuccess"></editUserModal>
		</view>
		<pageFooter :style="colorStyle"></pageFooter>
	</view>
</template>
<script>
let sysHeight = uni.getSystemInfoSync().statusBarHeight + 'px';
import { getUserInfo, setVisit, mpBindingPhone } from '@/api/user.js';
import { toLogin } from '@/libs/login.js';
import { mapGetters } from 'vuex';
// #ifdef H5
import Auth from '@/libs/wechat';
// #endif
const app = getApp();
import Routine from '@/libs/routine';
import colors from '@/mixins/color';
import pageFooter from '@/components/pageFooter/index.vue';
import { getCustomer } from '@/utils/index.js';
import editUserModal from '@/components/eidtUserModal/index.vue';
export default {
	components: {
		pageFooter,
		editUserModal
	},
	// computed: mapGetters(['isLogin','cartNum']),
	computed: {
		...mapGetters({
			cartNum: 'cartNum',
			isLogin: 'isLogin'
		})
	},
	mixins: [colors],
	data() {
		return {
			editModal: false, // 编辑头像信息
			orderMenu: [
				{
					img: 'icon-daifukuan',
					title: '待付款',
					url: '/pages/goods/order_list/index?status=0'
				},
				{
					img: 'icon-daifahuo',
					title: '待发货',
					url: '/pages/goods/order_list/index?status=1'
				},
				{
					img: 'icon-daishouhuo',
					title: '待收货',
					url: '/pages/goods/order_list/index?status=2'
				},
				{
					img: 'icon-daipingjia',
					title: '待评价',
					url: '/pages/goods/order_list/index?status=3'
				},
				{
					img: 'icon-a-shouhoutuikuan',
					title: '售后/退款',
					url: '/pages/users/user_return_list/index'
				}
			],
			isAuto: false, //没有授权的不会自动授权
			isShowAuth: false, //是否隐藏授权
			userInfo: {},
			showOrderMenu: true,
			sysHeight: sysHeight,
			// #ifdef H5 || MP
			pageHeight: '100%',
			// #endif
			// #ifdef APP-PLUS
			pageHeight: app.globalData.windowHeight,
			// #endif
			// #ifdef H5
			isWeixin: Auth.isWeixin(),
			//#endif
			footerSee: false
		};
	},
	onLoad(option) {
		uni.hideTabBar();
		let that = this;
		// #ifdef MP
		// 小程序静默授权
		if (!this.$store.getters.isLogin) {
			// Routine.getCode()
			// 	.then(code => {
			// 		Routine.silenceAuth(code).then(res => {
			// 			this.onLoadFun();
			// 		})
			// 	})
			// 	.catch(res => {
			// 		uni.hideLoading();
			// 	});
		}
		// #endif

		// #ifdef H5 || APP-PLUS
		// if (that.isLogin == false) {
		// 	toLogin();
		// }
		//获取用户信息回来后授权
		let cacheCode = this.$Cache.get('snsapi_userinfo_code');
		let res1 = cacheCode ? option.code != cacheCode : true;
		if (this.isWeixin && option.code && res1 && option.scope === 'snsapi_userinfo') {
			this.$Cache.set('snsapi_userinfo_code', option.code);
			Auth.auth(option.code)
				.then((res) => {
					this.getUserInfo();
				})
				.catch((err) => {});
		}
		// #endif
		// #ifdef APP-PLUS
		that.$set(that, 'pageHeight', app.globalData.windowHeight);
		// #endif
	},
	onShow: function () {
		let that = this;
		// #ifdef APP-PLUS
		uni.getSystemInfo({
			success: function (res) {
				that.pageHeight = res.windowHeight + 'px';
			}
		});
		// #endif
		if (that.isLogin) {
			this.getUserInfo();
			this.setVisit();
		}
	},
	onPullDownRefresh() {
		this.onLoadFun();
	},
	methods: {
		getWechatuserinfo() {
			//#ifdef H5
			Auth.isWeixin() && Auth.toAuth('snsapi_userinfo', '/pages/user/index');
			//#endif
		},
		editSuccess() {
			this.editModal = false;
			this.getUserInfo();
		},
		closeEdit() {
			this.editModal = false;
		},
		// 记录会员访问
		setVisit() {
			setVisit({
				url: '/pages/user/index'
			}).then((res) => {});
		},
		// 打开授权
		openAuto() {
			toLogin();
		},
		// 授权回调
		onLoadFun() {
			this.getUserInfo();
			this.getMyMenus();
			this.setVisit();
		},
		Setting: function () {
			uni.openSetting({
				success: function (res) {}
			});
		},
		// 授权关闭
		authColse: function (e) {
			this.isShowAuth = e;
		},
		// 绑定手机
		bindPhone() {
			uni.navigateTo({
				url: '/pages/users/user_phone/index'
			});
		},
		getphonenumber(e) {
			if (e.detail.errMsg == 'getPhoneNumber:ok') {
				Routine.getCode()
					.then((code) => {
						let data = {
							code,
							iv: e.detail.iv,
							encryptedData: e.detail.encryptedData
						};
						mpBindingPhone(data)
							.then((res) => {
								this.getUserInfo();
								this.$util.Tips({
									title: res.msg,
									icon: 'success'
								});
							})
							.catch((err) => {
								return this.$util.Tips({
									title: err
								});
							});
					})
					.catch((error) => {
						uni.hideLoading();
					});
			}
		},
		toggleOrderMenu() {
			this.showOrderMenu = !this.showOrderMenu;
		},
		contactTeacher() {
			uni.navigateTo({
				url: `/pages/course/contact_teacher`
			});
		},
		/**
		 * 获取个人用户信息
		 */
		getUserInfo: function () {
			let that = this;
			getUserInfo().then((res) => {
				that.userInfo = res.data;
				that.$store.commit('SETUID', res.data.uid);
				that.orderMenu.forEach((item, index) => {
					switch (item.title) {
						case '待付款':
							item.num = res.data.orderStatusNum.unpaid_count;
							break;
						case '待发货':
							item.num = res.data.orderStatusNum.unshipped_count;
							break;
						case '待收货':
							item.num = res.data.orderStatusNum.received_count;
							break;
						case '待评价':
							item.num = res.data.orderStatusNum.evaluated_count;
							break;
						case '售后/退款':
							item.num = res.data.orderStatusNum.refunding_count;
							break;
					}
				});
				uni.stopPullDownRefresh();
			});
		},
		//小程序授权api替换 getUserInfo
		getUserProfile() {
			toLogin();
		},
		// 编辑页面
		goEdit() {
			if (this.isLogin == false) {
				toLogin();
			} else {
				// #ifdef MP
				if (this.userInfo.is_default_avatar) {
					this.editModal = true;
					return;
				}
				// #endif
				uni.navigateTo({
					url: '/pages/users/user_info/index'
				});
			}
		},
		// 签到
		goSignIn() {
			uni.navigateTo({
				url: '/pages/users/user_sgin/index'
			});
		},

		goPages(url) {
			this.$util.JumpPath(url);
		},

		// goMenuPage
		goMenuPage(url, name) {
			if (this.isLogin) {
				if (url.indexOf('http') === -1) {
					// #ifdef H5 || APP-PLUS
					if (name && name === '客服接待') {
						// return window.location.href = `${location.origin}${url}`
						return uni.navigateTo({
							url: `/pages/annex/web_view/index?url=${location.origin}${url}`
						});
					} else if (name && name === '联系客服') {
						return getCustomer(url);
					} else if (name === '订单核销') {
						return uni.navigateTo({
							url: url
						});
						// return window.location.href = `${location.origin}${url}`
					}
					// #endif

					// #ifdef MP
					if (name && name === '联系客服') {
						return getCustomer(url);
					}
					if (url != '#' && url == '/pages/users/user_info/index') {
						uni.openSetting({
							success: function (res) {}
						});
					}
					// #endif
					uni.navigateTo({
						url: url,
						fail(err) {
							uni.switchTab({
								url: url
							});
						}
					});
				} else {
					uni.navigateTo({
						url: `/pages/annex/web_view/index?url=${url}`
					});
				}
			} else {
				// #ifdef MP
				this.openAuto();
				// #endif
				// #ifndef MP
				toLogin();
				// #endif
			}
		},
		goRouter(item) {
			var pages = getCurrentPages();
			var page = pages[pages.length - 1].$page.fullPath;
			if (item.link == page) return;
			uni.switchTab({
				url: item.link,
				fail(err) {
					uni.redirectTo({
						url: item.link
					});
				}
			});
		}
	}
};
</script>

<style lang="scss">
page,
body {
	height: 100%;
}


.cardVipB {
	background-color: #343a48;
	width: 100%;
	height: 124rpx;
	border-radius: 16rpx 16rpx 0 0;
	padding: 22rpx 30rpx 0 30rpx;
	margin-top: 16px;

	.left-box {
		.small {
			color: #f8d5a8;
			font-size: 28rpx;
			margin-left: 18rpx;
		}

		.pictrue {
			width: 40rpx;
			height: 45rpx;

			image {
				width: 100%;
				height: 100%;
			}
		}
	}

	.btn {
		color: #bbbbbb;
		font-size: 26rpx;
	}

	.icon-jiantou {
		margin-top: 6rpx;
	}
}

.cardVipA {
	position: absolute;
	background: url('~@/static/images/member.png') no-repeat;
	background-size: 100% 100%;
	width: 750rpx;
	height: 84rpx;
	bottom: -2rpx;
	left: 0;
	padding: 0 56rpx 0 135rpx;

	.left-box {
		font-size: 26rpx;
		color: #905100;
		font-weight: 400;
	}

	.btn {
		color: #905100;
		font-weight: 400;
		font-size: 24rpx;
	}

	.iconfont {
		font-size: 20rpx;
		margin: 4rpx 0 0 4rpx;
	}
}

.new-users {
	display: flex;
	flex-direction: column;
	height: 100%;
	background: #f6fbf7;

	.sys-head {
		position: relative;
		width: 100%;

		.bg {
			position: absolute;
			left: 0;
			top: 0;
			width: 100%;
			height: 100%;
			background: linear-gradient(180deg, rgba(246, 251, 247, 0.98) 0%, rgba(246, 251, 247, 0.78) 55%, rgba(246, 251, 247, 0) 100%);
		}

		.sys-title {
			z-index: 10;
			position: relative;
			height: 43px;
			text-align: center;
			line-height: 43px;
			font-size: 36rpx;
			color: rgba(17, 59, 46, 0.9);
		}
	}

	.head {
		.user-card {
			position: relative;
			width: 100%;
			margin: 18rpx auto 0;
			padding: 32rpx 28rpx;
			background: rgba(255, 255, 255, 0.94);
			border-radius: 24rpx;
			border: 1rpx solid rgba(17, 59, 46, 0.08);

			.user-info {
				z-index: 20;
				position: relative;
				display: flex;

				.headwear {
					position: absolute;
					right: -4rpx;
					top: -14rpx;
					width: 44rpx;
					height: 44rpx;

					image {
						width: 100%;
						height: 100%;
					}
				}

				.live {
					width: 28rpx;
					height: 28rpx;
					margin-left: 20rpx;
				}

				.bntImg {
					width: 120rpx;
					height: 120rpx;
					border-radius: 50%;
					text-align: center;
					line-height: 120rpx;
					background-color: unset;
					position: relative;

					.avatarName {
						font-size: 16rpx;
						color: #fff;
						text-align: center;
						background-color: rgba(0, 0, 0, 0.6);
						height: 37rpx;
						line-height: 37rpx;
						position: absolute;
						bottom: 0;
						left: 0;
						width: 100%;
					}
				}

				.avatar-box {
					position: relative;
					display: flex;
					align-items: center;
					justify-content: center;
					width: 120rpx;
					height: 120rpx;
					border-radius: 50%;

					&.on {
						.avatar {
							border: 2px solid rgba(86, 197, 150, 0.7);
							border-radius: 50%;
							box-sizing: border-box;
						}
					}
				}

				.avatar {
					position: relative;
					width: 120rpx;
					height: 120rpx;
					border-radius: 50%;
				}

				.info {
					flex: 1;
					display: flex;
					flex-direction: column;
					justify-content: space-between;
					margin-left: 20rpx;
					padding: 12rpx 0;

					.name {
						display: flex;
						align-items: center;
						color: rgba(17, 59, 46, 0.9);
						font-size: 31rpx;

						.nickname {
							max-width: 8em;
						}

						.vip {
							margin-left: 10rpx;

							image {
								width: 78rpx;
								height: 30rpx;
								display: block;
							}
						}
					}

					.num {
						display: flex;
						align-items: center;
						font-size: 26rpx;
						color: rgba(17, 59, 46, 0.65);

						image {
							width: 22rpx;
							height: 23rpx;
							margin-left: 20rpx;
						}
					}
				}
			}

			.message {
				align-self: flex-start;
				position: relative;
				margin-top: 8rpx;
				margin-right: 6rpx;

				.num {
					position: absolute;
					top: -8rpx;
					left: 18rpx;
					padding: 0 6rpx;
					height: 28rpx;
					border-radius: 12rpx;
					background-color: #fff;
					font-size: 18rpx;
					line-height: 28rpx;
					text-align: center;
					color: var(--view-theme);
				}

				.iconfont {
					font-size: 40rpx;
					color: rgba(17, 59, 46, 0.9);
				}
			}
		}

		.profile-list {
			margin: 18rpx 24rpx 0;
			border-radius: 20rpx;
			background: rgba(255, 255, 255, 0.94);
			border: 1rpx solid rgba(17, 59, 46, 0.08);
			overflow: hidden;
		}

		.profile-item {
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 26rpx 24rpx;
			font-size: 28rpx;
			color: rgba(17, 59, 46, 0.9);
			border-bottom: 1rpx solid rgba(17, 59, 46, 0.06);
		}

		.profile-item:last-child {
			border-bottom: 0;
		}

		.profile-right {
			display: flex;
			align-items: center;
			gap: 10rpx;
			color: rgba(17, 59, 46, 0.6);
			font-size: 26rpx;
		}

		.order-submenu {
			padding: 0 18rpx 20rpx;
			background: rgba(255, 255, 255, 0.94);
		}

		.order-submenu-grid {
			display: grid;
			grid-template-columns: repeat(5, 1fr);
			gap: 12rpx;
		}

		.order-submenu-item {
			display: flex;
			flex-direction: column;
			align-items: center;
			padding: 14rpx 0;
			border-radius: 14rpx;
			background: rgba(246, 251, 247, 0.9);
			border: 1rpx solid rgba(17, 59, 46, 0.08);
		}

		.order-submenu-icon {
			position: relative;
			text-align: center;
		}

		.order-submenu-icon .iconfont {
			font-size: 42rpx;
			color: var(--view-theme);
		}

		.order-submenu-text {
			margin-top: 8rpx;
			font-size: 24rpx;
			color: rgba(17, 59, 46, 0.78);
		}
	}

	.phone {
	color: #fff;
	background-color: var(--view-theme);
		border-radius: 15px;
		width: max-content;
		font-size: 24rpx;
		padding: 2px 10px;
		margin-top: 8rpx;
	}

	.order-status-num {
		min-width: 12rpx;
		background-color: #fff;
		color: var(--view-theme);
		border-radius: 15px;
		position: absolute;
		right: -14rpx;
		top: -15rpx;
		font-size: 20rpx;
		padding: 0 8rpx;
		border: 1px solid var(--view-theme);
	}

}

.new-users {
	padding-bottom: 0;
	padding-bottom: constant(safe-area-inset-bottom);
	padding-bottom: env(safe-area-inset-bottom);
}
</style>
