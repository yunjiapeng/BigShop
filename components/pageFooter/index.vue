<template>
	<view v-if="showTabBar" class="modern-footer-host">
		<view class="modern-footer">
			<view class="modern-footer-inner">
				<view class="nav-item" :class="{ active: isActive(navItems[0].link) }" @click="goRouter(navItems[0])">
					<text class="iconfont nav-icon" :class="navItems[0].icon"></text>
					<view class="nav-text">首页</view>
				</view>

				<view class="nav-item center" :class="{ active: isActive(navItems[1].link) }" @click="goRouter(navItems[1])">
					<view class="center-btn">
						<text class="center-icon">☀</text>
					</view>
					<view class="nav-text">班级</view>
				</view>

				<view class="nav-item" :class="{ active: isActive(navItems[2].link) }" @click="goRouter(navItems[2])">
					<text class="iconfont nav-icon" :class="navItems[2].icon"></text>
					<view class="nav-text">我的</view>
				</view>
			</view>
		</view>
		<view class="modern-footer-spacer" :style="{ height: `${footerHeight}px` }"></view>
		<view class="safe-area-inset-bottom"></view>
	</view>
</template>

<script>
import { mapGetters } from 'vuex';
import { getNavigation } from '@/api/public.js';
import { getDiyVersion } from '@/api/api.js';
export default {
	name: 'pageFooter',
	props: {
		isTabBar: {
			type: Boolean,
			default: true
		},
		configData: {
			type: Object,
			default: () => {}
		}
	},
	computed: {
		...mapGetters(['isLogin'])
	},
	watch: {
		configData(newVal) {
			if (newVal) this.setNavigationInfo(newVal);
		}
	},
	created() {
		this.syncActiveRoute();
	},
	mounted() {
		this.navigationInfo();
		this.updateFooterHeight();
	},
	data() {
		return {
			newData: {},
			activeRouter: '',
			showTabBar: true,
			footerHeight: 0,
			navItems: [
				{ key: 'home', name: '首页', link: '/pages/index/index', icon: 'icon-shouye1' },
				{ key: 'class', name: '班级', link: '/pages/goods_cate/goods_cate', icon: 'icon-fenlei' },
				{ key: 'mine', name: '我的', link: '/pages/user/index', icon: 'icon-wode' }
			]
		};
	},
	methods: {
		updateFooterHeight() {
			try {
				this.footerHeight = uni.upx2px(96);
			} catch (e) {
				this.footerHeight = 48;
			}
		},
		syncActiveRoute() {
			let routes = getCurrentPages();
			let cur = routes[routes.length - 1];
			let fullPath = '';
			if (cur && cur.$page && cur.$page.fullPath) fullPath = cur.$page.fullPath;
			else if (cur && cur.route) fullPath = '/' + cur.route;
			this.activeRouter = (fullPath || '').split('?')[0];
		},
		isActive(link) {
			return (this.activeRouter || '').split('?')[0] === link;
		},
		setNavigationInfo(data) {
			if (this.isTabBar) {
				this.newData = data;
				const hasTabVal = !!(data && data.effectConfig && typeof data.effectConfig.tabVal !== 'undefined');
				const visible = hasTabVal ? !!data.effectConfig.tabVal : true;
				this.showTabBar = visible;
				const pdHeight = data && data.topConfig && data.bottomConfig ? data.topConfig.val + data.bottomConfig.val : 30;
				this.$emit('newDataStatus', visible, pdHeight);
				if (visible) {
					uni.hideTabBar();
				} else {
					uni.showTabBar();
				}
			}
		},
		getNavigationInfo() {
			getNavigation().then((res) => {
				uni.setStorageSync('diyVersionNav', res.data);
				this.setNavigationInfo(res.data);
			});
		},
		navigationInfo() {
			let footerNavigation = uni.getStorageSync('footerNavigation');
			if (footerNavigation) {
				getDiyVersion(0).then((res) => {
					let diyVersion = uni.getStorageSync('diyVersionNav');
					if (res.data.version + '0' === diyVersion) {
						this.setNavigationInfo(footerNavigation);
					} else {
						uni.setStorageSync('diyVersionNav', res.data.version + '0');
						this.getNavigationInfo();
					}
				});
			} else {
				this.getNavigationInfo();
			}
		},
		goRouter(item) {
			this.syncActiveRoute();
			var pages = getCurrentPages();
			var page = pages[pages.length - 1].$page.fullPath;
			if (item.link == page) return;
			if (item.link == '/pages/short_video/appSwiper/index' || item.link == '/pages/short_video/nvueSwiper/index') {
				//#ifdef APP
				item.link = '/pages/short_video/appSwiper/index';
				//#endif
				//#ifndef APP
				item.link = '/pages/short_video/nvueSwiper/index';
				//#endif
			}
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

<style scoped lang="scss">
.safe-area-inset-bottom {
	height: 0;
	height: constant(safe-area-inset-bottom);
	height: env(safe-area-inset-bottom);
}

.modern-footer-host {
	position: relative;
}

.modern-footer {
	position: fixed;
	left: 0;
	right: 0;
	bottom: 0;
	z-index: 999;
	padding: 0 24rpx;
	padding-bottom: calc(env(safe-area-inset-bottom));
}

.modern-footer-inner {
	position: relative;
	height: 96rpx;
	display: flex;
	align-items: flex-end;
	justify-content: space-between;
	gap: 12rpx;
	padding: 10rpx 10rpx 12rpx;
	border-radius: 32rpx;
	overflow: visible;
	background: #CDEBCB;
	border: 1rpx solid #B7DBB6;
	box-shadow: 0 16rpx 36rpx rgba(96, 140, 96, 0.22);
}

.modern-footer-inner::before {
	content: '';
	position: absolute;
	left: 0;
	right: 0;
	bottom: 0;
	height: 110rpx;
	transform: translateY(12rpx);
	border-radius: 32rpx;
	background: #DDF3DD;
	pointer-events: none;
}

.nav-item {
	flex: 1;
	display: flex;
	flex-direction: column;
	align-items: center;
	justify-content: flex-end;
	gap: 4rpx;
	padding: 8rpx 0;
	min-width: 0;
	transition: transform 220ms cubic-bezier(0.2, 0.85, 0.2, 1), color 220ms ease, opacity 220ms ease;
}

.nav-icon {
	font-size: 52rpx;
	line-height: 1;
	color: #000000;
	transition: transform 220ms cubic-bezier(0.2, 0.85, 0.2, 1), color 220ms ease;
	z-index: 2;
}

.nav-text {
	font-size: 22rpx;
	line-height: 26rpx;
	color: #000000;
	transition: color 220ms ease, opacity 220ms ease;
	white-space: nowrap;
	z-index: 2;
}

.nav-item.active .nav-icon,
.nav-item.active .nav-text {
	color: #000000;
}

.nav-item:active {
	transform: scale(0.96);
}

.nav-item.center {
	position: relative;
	flex: 1;
	padding-top: 0;
	justify-content: flex-end;
}

.center-btn {
	width: 64rpx;
	height: 64rpx;
	border-radius: 32rpx;
	display: flex;
	align-items: center;
	justify-content: center;
	background: #FFE3D6;
	box-shadow: 0 14rpx 26rpx rgba(255, 108, 74, 0.28);
	border: 2rpx solid #FFD4C4;
	transform: translateY(0);
	transition: transform 260ms cubic-bezier(0.2, 0.85, 0.2, 1), box-shadow 260ms ease, filter 260ms ease;
}

.center-icon {
	font-size: 40rpx;
	line-height: 1;
	color: #FF6A3D;
}

.nav-item.center:active .center-btn {
	transform: scale(0.96);
	filter: brightness(0.98);
}

.nav-item.center .nav-text {
	margin-top: 6rpx;
	font-size: 22rpx;
	color: #000000;
}

.nav-item.center.active .nav-text {
	color: #000000;
}

.modern-footer-spacer {
	width: 100%;
}
</style>
