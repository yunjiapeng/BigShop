<template>
	<!-- 首页 -->
	<view v-if="pageShow" class="page" :class="bgTabVal == 2 ? 'fullsize noRepeat' : bgTabVal == 1 ? 'repeat ysize' : 'noRepeat ysize'" :style="[pageStyle]">
		<view v-if="!errorNetwork" :style="colorStyle">
			<!-- #ifdef MP -->
			<view class="fixed z-1000" :style="[appletStyle]" v-if="myApplet">
				<view class="myApplet w-324 h-62 text-center rd-12rpx lh-62rpx fs-24 bg--w111-fff text-w111-303133">
					点击添加到我的小程序
					<text class="iconfont icon-ic_close2 text--w111-ccc ml-16" @click="myApplet = false"></text>
				</view>
			</view>
			<!-- #endif -->
			<image class="home-logo" :src="logoSrc" @error="logoError" mode="widthFix" :style="{ width: logoWidth + 'px' }"></image>
			<!-- 轮播搜索 -->
			<homeComb v-if="showHomeComb" :dataConfig="homeCombData" :belongIndex="belongIndex" @bindSortId="bindSortId" :isScrolled="isScrolled" @storeTap="storeTap"></homeComb>
			<view class="notice-and-entrances">
				<view class="notice-wrap" v-if="newsItem">
					<news :dataConfig="newsItem"></news>
				</view>
				<view class="entrance-row">
					<view class="entrance-item" @click="openModuleArticle(3)">
						<image class="entrance-icon-img" :src="classIcon1" mode="widthFix"></image>
						<view class="entrance-text">课程介绍</view>
					</view>
					<view class="entrance-item" @click="openModuleArticle(4)">
						<image class="entrance-icon-img" :src="classIcon2" mode="widthFix"></image>
						<view class="entrance-text">教育理念</view>
					</view>
					<view class="entrance-item" @click="openModuleArticle(5)">
						<image class="entrance-icon-img" :src="classIcon3" mode="widthFix"></image>
						<view class="entrance-text">师资团队</view>
					</view>
					<view class="entrance-item" @click="openModuleArticle(6)">
						<image class="entrance-icon-img" :src="classIcon4" mode="widthFix"></image>
						<view class="entrance-text">创始人故事</view>
					</view>
				</view>
				<view class="virtual-card" @click="openVirtual" :style="{ backgroundImage: 'url(/picture720/p_1.jpg)' }">
					<view class="virtual-title">虚拟导览</view>
				</view>
			</view>
			<view class="index">
				<!-- 自定义样式 -->
				<block v-for="(item, index) in filteredStyleConfig" :key="index">
					<userInfor v-if="item.name == 'userInfor'" :dataConfig="item" @changeLogin="changeLogin"></userInfor>
					<newVip v-if="item.name == 'newVip'" :dataConfig="item"></newVip>
					<view v-if="item.name == 'articleList'" class="modal-target">
						<articleList :dataConfig="item" :disableNavigate="true" :excludeIds="homepageExcludeIds" @open="openArticleModal"></articleList>
					</view>
					<bargain v-if="item.name == 'bargain'" :dataConfig="item" @changeBarg="changeBarg"></bargain>
					<blankPage v-if="item.name == 'blankPage'" :dataConfig="item"></blankPage>
					<combination v-if="item.name == 'combination'" :dataConfig="item"></combination>
					<!-- 客户服务 -->
					<customerService v-if="item.name == 'customerService'" :dataConfig="item"></customerService>
					<guide v-if="item.name == 'guide'" :dataConfig="item"></guide>
					<!-- 直播模块 -->
					<!-- #ifdef  MP-WEIXIN -->
					<liveBroadcast v-if="item.name == 'liveBroadcast'" :dataConfig="item"></liveBroadcast>
					<!-- #endif -->
					<!-- 图片库 -->
					<pictureCube v-if="item.name == 'pictureCube'" :dataConfig="item"></pictureCube>
					<!-- 促销列表 -->
					<promotionList
						ref="promotionLists"
						v-if="item.name == 'promotionList'"
						:dataConfig="item"
						:productVideoStatus="product_video_status"
						:positionTop="positionTop"
					></promotionList>
					<seckill v-if="item.name == 'seckill'" :dataConfig="item"></seckill>
					<!-- 轮播图-->
					<swiperBg v-if="item.name == 'swiperBg'" :dataConfig="item"></swiperBg>
					<swipers v-if="item.name == 'swipers'" :dataConfig="item"></swipers>
					<!-- 顶部选项卡 -->

					<!-- 标题 -->
					<titles v-if="item.name == 'titles'" :dataConfig="item"></titles>
					<ranking v-if="item.name == 'ranking'" :dataConfig="item"></ranking>
					<presale v-if="item.name == 'presale'" :dataConfig="item"></presale>
					<pointsMall v-if="item.name == 'pointsMall'" :dataConfig="item"></pointsMall>
					<!-- #ifndef APP -->
					<richText v-if="item.name == 'richText'" :dataConfig="item"></richText>
					<videos v-if="item.name == 'videos'" :dataConfig="item"></videos>
					<!-- #endif -->
					<hotspot v-if="item.name == 'hotspot'" :dataConfig="item"></hotspot>
					<follow v-if="item.name == 'follow'" :dataConfig="item"></follow>
				</block>
				<view class="site-config-placeholder">
					{{ site_config }}
				</view>
				<!-- #ifndef APP-PLUS -->
				<view v-if="configData && configData.record_No" class="site-config" @click="goICP(configData.icp_url)">{{ configData.record_No }}</view>
				<view class="site-config" v-if="configData && configData.network_security" @click="goICP(configData.network_security_url)">
					<image class="ban" src="/static/images/beian.png" alt="" srcset="" />
					{{ configData.network_security }}
				</view>
				<!-- #endif -->
				<view class="pb-safe" :style="[pdHeights]" v-if="isFooter"></view>
				<pageFooter :configData="footerConfigData" @newDataStatus="newDataStatus"></pageFooter>
				<view v-show="modalVisible" class="modal-mask" :class="{ 'modal-mask-show': modalActive }" @click="closeModal" @touchmove.stop.prevent>
					<view class="modal-panel" :class="{ 'modal-panel-show': modalActive }" @click.stop>
						<view class="modal-header">
							<view class="modal-title">{{ modalTitle }}</view>
							<text class="modal-close" @click="closeModal">×</text>
						</view>
						<view class="modal-body">
							<view v-if="modalType === 'virtual'">
								<swiper class="pano-swiper" :current="panoIndex" :circular="true" :duration="400" @change="panoChange">
									<swiper-item v-for="(src, idx) in panoList" :key="idx">
										<image class="pano-image" :src="src" mode="aspectFill"></image>
									</swiper-item>
								</swiper>
								<view class="pano-indicator" v-if="panoList.length">{{ panoIndex + 1 }} / {{ panoList.length }}</view>
							</view>
							<view v-else-if="modalType === 'news' && newsItem" class="modal-section">
								<news :dataConfig="newsItem"></news>
							</view>
							<view v-else-if="modalType === 'article'" class="modal-section">
								<view v-if="articleLoading" class="modal-text">加载中...</view>
								<view v-else-if="articleDetail" class="article-detail">
									<view class="article-title">{{ articleDetail.title }}</view>
									<view class="article-meta">
										<view class="article-label">{{ articleDetail.catename }}</view>
										<view class="article-meta-item">{{ articleDetail.add_time }}</view>
										<view class="article-meta-item"><text class="iconfont icon-liulan"></text>{{ articleDetail.visit }}</view>
									</view>
									<view class="article-content" v-if="articleDescription">
										<!-- #ifndef APP-PLUS -->
										<parser :html="articleDescription" :tag-style="articleTagStyle"></parser>
										<!-- #endif -->
										<!-- #ifdef APP-PLUS -->
										<view v-html="articleDescription"></view>
										<!-- #endif -->
									</view>
								</view>
								<view v-else class="modal-text">暂无内容</view>
							</view>
							<view v-else class="modal-text">{{ modalTitle }}</view>
						</view>
					</view>
				</view>
			</view>
			<!-- #ifdef APP -->
			<app-update ref="appUpdate" :force="true" :tabbar="false"></app-update>
			<!-- #endif -->
		</view>
		<view v-else>
			<view class="error-network">
				<image :src="imgHost + '/statics/images/error-network.gif'"></image>
				<view class="title">{{ $t(`网络连接断开`) }}</view>
				<view class="con">
					<view class="label">{{ $t(`请检查情况`) }}：</view>
					<view class="item">· {{ $t(`在设置中是否已开启网络权限`) }}</view>
					<view class="item">· {{ $t(`当前是否处于弱网环境`) }}</view>
					<view class="item">· {{ $t(`版本是否过低，升级试试吧`) }}</view>
				</view>
				<view class="btn" @click="reconnect">{{ $t(`重新连接`) }}</view>
			</view>
		</view>
	</view>
</template>

<script>
const app = getApp();
import colors from '@/mixins/color';
import logoPng from '@/logo/logo.png';
import classIcon1 from '@/logo/class_1.png';
import classIcon2 from '@/logo/class_2.png';
import classIcon3 from '@/logo/class_3.png';
import classIcon4 from '@/logo/class_4.png';
import couponWindow from '@/components/couponWindow/index';
import { getCouponV2, getCouponNewUser, getCrmebCopyRight, getArticleDetails } from '@/api/api.js';
import { getShare } from '@/api/public.js';
import userInfor from './components/userInfor';
import homeComb from './components/homeComb';
import newVip from './components/newVip';
import headerSerch from './components/headerSerch';
import swipers from './components/swipers';
import coupon from './components/coupon';
import articleList from './components/articleList';
import bargain from './components/bargain';
import blankPage from './components/blankPage';
import combination from './components/combination';
import customerService from './components/customerService';
import goodList from './components/goodList';
import guide from './components/guide';
import liveBroadcast from './components/liveBroadcast';
import menus from './components/menus';
import news from './components/news';
import pictureCube from './components/pictureCube';
import promotionList from './components/promotionList';
import seckill from './components/seckill';
import swiperBg from './components/swiperBg';
import tabNav from './components/tabNav';
import titles from './components/titles';
import ranking from './components/ranking';
import presale from './components/presale';
import pointsMall from './components/pointsMall';
// #ifndef APP
import richText from './components/richText';
import videos from './components/videos';
// #endif
import signIn from './components/signIn';
import hotspot from './components/hotspot';
import follow from './components/follow';
import waterfallsFlow from '@/components/WaterfallsFlow/WaterfallsFlow.vue';
import emptyPage from '@/components/emptyPage.vue';
import parser from '@/components/jyf-parser/jyf-parser';
// #ifdef MP
import { getTempIds } from '@/api/api.js';
import { SUBSCRIBE_MESSAGE } from '@/config/cache';
// #endif
import { mapGetters, mapMutations } from 'vuex';
import { getDiy, getDiyVersion } from '@/api/api.js';
import { getCartCounts } from '@/api/order.js';
import { getCategoryList, getProductslist } from '@/api/store.js';
import { goShopDetail } from '@/libs/order.js';
import { toLogin } from '@/libs/login.js';
import { HTTP_REQUEST_URL } from '@/config/app';
import pageFooter from '@/components/pageFooter/index.vue';
import Loading from '@/components/Loading/index.vue';
import Cache from '@/utils/cache';
import appUpdate from '@/components/update/app-update.vue';

export default {
	computed: {
		// #ifdef MP
		appletStyle() {
			return {
				top: this.getHeight.menuButtonInfo.bottom + 8 + 'px',
				right: '10px'
			};
		},
		// #endif
		pageStyle() {
			return {
				backgroundColor: this.bgColor,
				backgroundImage: this.bgPic ? `url(${this.bgPic})` : '',
				minHeight: this.windowHeight + 'px'
			};
		},
		pdHeights() {
			let H = `${this.pdHeight * 2 + 100}rpx`;
			return {
				height: this.isFooter ? H : '100rpx'
			};
		},
		newsItem() {
			return (this.styleConfig || []).find((item) => item.name === 'news');
		},
		articleListItem() {
			return (this.styleConfig || []).find((item) => item.name === 'articleList');
		},
		filteredStyleConfig() {
			return (this.styleConfig || []).filter((item) => item.name !== 'news');
		},
		modalTitle() {
			if (this.modalType === 'virtual') return '虚拟导览';
			if (this.modalType === 'news') return '公告';
			if (this.modalType === 'article') return this.articleDetail && this.articleDetail.title ? this.articleDetail.title : '图文专栏';
			if (this.modalType === 'forest') return '课程介绍';
			if (this.modalType === 'international') return '教育理念';
			if (this.modalType === 'parenting') return '师资团队';
			if (this.modalType === 'transfer') return '创始人故事';
			return '';
		},
		...mapGetters(['isLogin', 'uid', 'cartNum'])
	},
	mixins: [colors],
	components: {
		Loading,
		pageFooter,
		couponWindow,
		homeComb,
		newVip,
		userInfor,
		headerSerch,
		swipers,
		coupon,
		articleList,
		bargain,
		blankPage,
		combination,
		customerService,
		goodList,
		guide,
		liveBroadcast,
		menus,
		pictureCube,
		news,
		promotionList,
		seckill,
		swiperBg,
		tabNav,
		titles,
		ranking,
		presale,
		pointsMall,
		// #ifndef APP
		richText,
		videos,
		// #endif
		signIn,
		hotspot,
		follow,
		waterfallsFlow,
		emptyPage,
		parser,
		// #ifdef APP
		appUpdate
		// #endif
	},
	data() {
		return {
			logoSrc: logoPng,
			classIcon1,
			classIcon2,
			classIcon3,
			classIcon4,
			homepageExcludeIds: [3, 4, 5, 6],
			modalVisible: false,
			modalActive: false,
			modalTimer: null,
			modalType: '',
			articleDetail: null,
			articleDescription: '',
			articleLoading: false,
			articleTagStyle: {
				img: 'width:100%;display:block;',
				table: 'width:100%',
				video: 'width:100%'
			},
			panoList: [],
			panoIndex: 0,
			logoTry: 0,
			logoWidth: 0,
			styleConfig: [],
			loading: false,
			loadend: false,
			loadTitle: '下拉加载更多', //提示语
			page: 1,
			limit: this.$config.LIMIT,
			numConfig: 0,
			code: '',
			isCouponShow: false,
			couponObj: {},
			couponObjs: {
				show: false
			},
			shareInfo: {},
			sortList: '',
			sortAll: [],
			goodPage: 1,
			goodList: [],
			sid: 0,
			curSort: 0,
			sortMpTop: 0,
			loaded: false,
			loading: false,
			domOffsetTop: 50,
			// #ifdef APP-PLUS || MP
			isFixed: true,
			// #endif
			// #ifdef H5
			isFixed: false,
			// #endif
			site_config: '',
			errorNetwork: false, // 是否断网
			isHeaderSerch: false,
			showHomeComb: false,
			showCateNav: false,
			homeCombData: {},
			headerSerchCombData: {},
			cateNavData: {},
			footerConfigData: {},
			bgColor: '',
			bgPic: '',
			bgTabVal: '',
			pageShow: true,
			windowHeight: 0,
			imgHost: HTTP_REQUEST_URL,
			isShowAuth: false,
			isScrolled: false,
			product_video_status: false,
			confirm_video_status: false,
			positionTop: 0,
			isFooter: false,
			pdHeight: 0, //自定义底部导航上下边距和
			entryData: {
				store_id: '',
				latitude: '',
				longitude: '',
				select_store_id: ''
			},
			goodsIndex: [],
			promotionIndex: [],
			belongIndex: 0, // 进店规则归属门店排序位置；
			isBelongStore: false, //判断是否为归属门店；
			getHeight: this.$util.getWXStatusHeight(),
			myApplet: true,
			configData: Cache.get('BASIC_CONFIG')
		};
	},
	onLoad(options) {
		let that = this;
		uni.getSystemInfo({
			success: function (res) {
				that.logoWidth = res.windowWidth / 3;
			}
		});
		uni.hideTabBar();
		that.getOptions(options);
		this.$nextTick(function () {
			uni.getSystemInfo({
				success: function (res) {
					that.windowHeight = res.windowHeight;
				}
			});
		});
		const { state, scope } = options;
		this.diyData();
		// #ifdef H5
		this.setOpenShare();
		// #endif
		// #ifdef MP
		this.getTempIds();
		// #endif
		getShare().then((res) => {
			this.shareInfo = res.data;
		});
		this.getCopyRight();
		this.$eventHub.$on('confirm_video_status', () => {
			if (this.confirm_video_status) {
				return;
			}
			this.confirm_video_status = true;
			let flag = true;
			// #ifdef H5
			flag = window.self == window.top;
			// #endif
			if (!flag) {
				return;
			}
			uni.showModal({
				content: '当前使用移动网络，是否继续播放视频？',
				success: (res) => {
					if (res.confirm) {
						// 监听
						this.SET_AUTOPLAY(true);
						this.$eventHub.$emit('product_video_observe');
					}
				}
			});
		});
	},
	onUnload() {
		// 清除监听
		uni.$off('activeFn');
	},
	watch: {
		isLogin: {
			deep: true, //深度监听设置为 true
			handler: function (newV, oldV) {
				// 优惠券弹窗
				var newDates = new Date().toLocaleDateString();
				if (newV) {
					try {
						var oldDate = uni.getStorageSync('oldDate') || '';
					} catch {}
					if (oldDate != newDates) {
						this.getCoupon();
					}
				}
			}
		}
	},
	onShow() {
		uni.removeStorageSync('form_type_cart');
		// 优惠券弹窗
		if (this.isLogin) {
			this.getCoupon();
			this.getCartNum();
		}
		// #ifdef MP
		if (wx.canIUse('checkIsAddedToMyMiniProgram')) {
			this.checkMyApplet();
		} else {
			this.myApplet = true;
		}
		// #endif
	},
	onPullDownRefresh() {
		this.diyData();
		uni.stopPullDownRefresh();
	},
	methods: {
		logoError() {
			this.logoTry += 1;
			if (this.logoTry === 1) {
				this.logoSrc = '/logo/logo.png';
			} else if (this.logoTry === 2) {
				this.logoSrc = '../../logo/logo.png';
			}
		},
		...mapMutations(['SET_AUTOPLAY', 'SET_NEARBY']),
		checkMyApplet() {
			wx.checkIsAddedToMyMiniProgram({
				success: (res) => {
					if (res.added) {
						this.myApplet = false;
					} else {
						this.myApplet = true;
					}
				},
				fail: () => {
					this.myApplet = true;
				}
			});
		},
		getCartNum: function () {
			getCartCounts()
				.then((res) => {
					this.$store.commit('indexData/setCartNum', res.data.count + '');
					let cartNum = res.data.count;
					if (cartNum > 0) {
						uni.setTabBarBadge({
							index: 3,
							text: cartNum > 99 ? '99+' : cartNum + ''
						});
					} else {
						uni.hideTabBarRedDot({
							index: 3
						});
					}
				})
				.catch((err) => {
					return this.$util.Tips({
						title: err.msg
					});
				});
		},
		storeTap(id) {
			this.entryData.select_store_id = id;
			this.entryData.store_id = '';
			uni.removeStorageSync('rulesStoreId');
		},
		getCopyRight() {
			getCrmebCopyRight()
				.then((res) => {
					let data = res.data;
					uni.setStorageSync('wechatStatus', data.wechat_status);
					if (!data.copyrightContext && !data.copyrightImage) {
						data.copyrightImage = '/static/images/support.png';
					}
					uni.setStorageSync('copyNameInfo', data.copyrightContext);
					uni.setStorageSync('copyImageInfo', data.copyrightImage);
					// #ifdef MP
					uni.setStorageSync(
						'MPSiteData',
						JSON.stringify({
							site_logo: data.site_logo,
							site_name: data.site_name
						})
					);
					// #endif
				})
				.catch((err) => {
					return this.$util.Tips({
						title: err.msg
					});
				});
		},
		getOptions(options) {
			let that = this;
			// #ifdef MP
			if (options.scene) {
				let value = that.$util.getUrlParams(decodeURIComponent(options.scene));
				//记录推广人uid
				if (value.spid) app.globalData.spid = value.spid;
			}
			// #endif
			if (options.spid) app.globalData.spid = options.spid;
		},
		// 重新链接
		reconnect() {
			this.diyData();
			getShare().then((res) => {
				this.shareInfo = res.data;
			});
		},
		goICP(url) {
			// #ifdef H5
			window.open(url);
			// #endif
			// #ifdef MP
			uni.navigateTo({
				url: `/pages/annex/web_view/index?url=${url}`
			});
			// #endif
		},
		bindHeighta(data) {
			// #ifdef APP-PLUS
			this.sortMpTop = data.top + data.height;
			// #endif
		},
		bindHeight(data) {
			uni.hideLoading();
			this.domOffsetTop = data.top;
		},
		// 去商品详情
		goGoodsDetail(item) {
			goShopDetail(item, this.uid).then((res) => {
				uni.navigateTo({
					url: `/pages/goods_details/index?id=${item.id}`
				});
			});
		},
		// 分类点击
		changeSort(item, index) {
			if (this.curSort == index) return;
			this.curSort = index;
			this.sid = item.id;
			this.goodList = [];
			this.goodPage = 1;
			this.loaded = false;
			this.getGoodsList();
		},
		/**
			 * @param data {
				classPage: 0 分类id
				microPage: 0 微页面id
				type: 1   0 微页面 1 商品分类
			 }*/
		bindSortId(data) {
			this.styleConfig = [];
			if (data.type == 1) {
				this.getProductList(data.classPage);
			} else {
				this.sortList = [];
				this.getMicroPage(data.microPage, true);
			}
		},
		/**
		 * 获取DIY
		 * @param {number} id
		 * @param {boolean} type 区分是否是微页面
		 */
		getMicroPage(id, type) {
			let that = this;
			that.styleConfig = [];
			uni.showLoading({
				title: '加载中...'
			});
			getDiy(id)
				.then((res) => {
					uni.hideLoading();
					let data = res.data;
					let diyArr = that.objToArr(res.data.value);
					diyArr = diyArr.filter((item) => !item.isHide);
					diyArr.forEach((item, index) => {
						if (['headerSerch', 'homeComb'].includes(item.name)) {
							diyArr.splice(index, 1);
						}
					});
					this.styleConfig = diyArr;
				})
				.catch((err) => {
					return that.$util.Tips({
						title: err
					});
					uni.hideLoading();
				});
		},
		getProductList(data) {
			let tempObj = '';
			this.curSort = 0;
			this.loaded = false;
			if (this.sortAll.length > 0) {
				this.sortAll.forEach((el, index) => {
					if (el.id == data) {
						this.$set(this, 'sortList', el);
						this.sid = el.children.length ? el.children[0].id : '';
					}
				});
				this.goodList = [];
				this.goodPage = 1;
				this.$nextTick(() => {
					if (this.sortList != '') this.getGoodsList();
				});
			} else {
				getCategoryList().then((res) => {
					this.sortAll = res.data;
					res.data.forEach((el, index) => {
						if (el.id == data) {
							this.sortList = el;
							this.sid = el.children.length ? el.children[0].id : '';
						}
					});
					this.goodList = [];
					this.goodPage = 1;

					this.$nextTick(() => {
						if (this.sortList != '') this.getGoodsList();
					});
				});
			}
		},
		// 商品列表
		getGoodsList() {
			if (this.loading || this.loaded) return;
			this.loading = true;
			getProductslist({
				sid: this.sid,
				keyword: '',
				priceOrder: '',
				salesOrder: '',
				news: 0,
				page: this.goodPage,
				limit: 10,
				cid: this.sortList.id
			}).then((res) => {
				this.loading = false;
				this.loaded = res.data.length < 10;
				this.goodPage++;
				this.goodList = this.goodList.concat(res.data);
			});
		},
		// 新用户优惠券
		getNewCoupon() {
			const oldUser = uni.getStorageSync('oldUser') || 0;
			if (!oldUser) {
				getCouponNewUser().then((res) => {
					const { data } = res;
					if (data.show) {
						if (data.list.length) {
							this.isCouponShow = true;
							this.couponObj = data;
							uni.setStorageSync('oldUser', 1);
						}
					} else {
						uni.setStorageSync('oldUser', 1);
					}
				});
			}
		},
		// 优惠券弹窗
		getCoupon() {
			const tagDate = uni.getStorageSync('tagDate') || '',
				nowDate = new Date().toLocaleDateString();
			if (tagDate === nowDate) {
				this.getNewCoupon();
			} else {
				getCouponV2().then((res) => {
					const { data } = res;
					if (data.list.length) {
						this.isCouponShow = true;
						this.couponObj = data;
						uni.setStorageSync('tagDate', new Date().toLocaleDateString());
					} else {
						this.getNewCoupon();
					}
				});
			}
		},
		// 优惠券弹窗关闭
		couponClose() {
			this.isCouponShow = false;
			if (!uni.getStorageSync('oldUser')) {
				this.getNewCoupon();
			}
		},
		onLoadFun() {
			this.isShowAuth = false;
		},
		// #ifdef H5
		// 获取url后面的参数
		getQueryString(name) {
			var reg = new RegExp('(^|&)' + name + '=([^&]*)(&|$)', 'i');
			var reg_rewrite = new RegExp('(^|/)' + name + '/([^/]*)(/|$)', 'i');
			var r = window.location.search.substr(1).match(reg);
			var q = window.location.pathname.substr(1).match(reg_rewrite);
			if (r != null) {
				return unescape(r[2]);
			} else if (q != null) {
				return unescape(q[2]);
			} else {
				return null;
			}
		},
		// #endif

		// #ifdef MP
		getTempIds() {
			let messageTmplIds = wx.getStorageSync(SUBSCRIBE_MESSAGE);
			if (!messageTmplIds) {
				getTempIds().then((res) => {
					if (res.data) wx.setStorageSync(SUBSCRIBE_MESSAGE, JSON.stringify(res.data));
				});
			}
		},
		// #endif
		// 对象转数组
		objToArr(data) {
			let obj = Object.keys(data).sort();
			let m = obj.map((key) => data[key]);
			return m;
		},
		setDiyData(data) {
			this.errorNetwork = false;
			if (data.is_bg_color) {
				this.bgColor = data.color_picker;
			}
			if (data.is_bg_pic) {
				this.bgPic = data.bg_pic;
				this.bgTabVal = data.bg_tab_val;
			}
			this.pageShow = data.is_show;
			uni.setNavigationBarTitle({
				title: data.title
			});
			let temp = [];
			let goodsIndex = [];
			let promotionIndex = [];
			let lastArr = this.objToArr(data.value);
			lastArr.forEach((item, index, arr) => {
				if (item.name == 'pageFoot') {
					this.footerConfigData = item;
				}
				if (item.name === 'homeComb' && !item.isHide) {
					this.showHomeComb = true;
					this.homeCombData = item;
					if (item.searchConfig.tabVal) {
						this.positionTop = uni.getSystemInfoSync().statusBarHeight + 43;
					}
				}
				if (item.name == 'headerSerch' && !item.isHide) {
					this.isHeaderSerch = true;
					this.headerSerchCombData = item;
				}
				if (item.name == 'tabNav' && !item.isHide) {
					this.showCateNav = true;
					this.cateNavData = item;
				}
				if (item.name == 'goodList' && !item.isHide) {
					goodsIndex.push(index);
				}
				if (item.name == 'promotionList' && !item.isHide) {
					promotionIndex.push(index);
				}
				if (!item.isHide) {
					temp.push(item);
				}
			});

			function sortNumber(a, b) {
				return a.timestamp - b.timestamp;
			}
			temp.sort(sortNumber);
			this.styleConfig = temp;
			this.goodsIndex = goodsIndex;
			this.promotionIndex = promotionIndex;
		},
		getDiyData() {
			getDiy(0)
				.then((res) => {
					uni.setStorageSync('diyData', JSON.stringify(res.data));
					this.setDiyData(res.data);
				})
				.catch((error) => {
					// #ifdef APP-PLUS
					if (error.status) {
						uni.hideLoading();
						if (this.errorNetwork) {
							uni.showToast({
								title: '请开启网络连接',
								icon: 'none',
								duration: 2000
							});
						}
						this.errorNetwork = true;
					}
					// #endif
				});
		},
		diyData() {
			let diyData = uni.getStorageSync('diyData');
			if (diyData) {
				getDiyVersion(0).then((res) => {
					let diyVersion = uni.getStorageSync('diyVersion');
					if (res.data.version + '0' === diyVersion) {
						this.setDiyData(JSON.parse(diyData));
					} else {
						uni.setStorageSync('diyVersion', res.data.version + '0');
						this.getDiyData();
					}
				});
			} else {
				this.getDiyData();
			}
		},

		changeLogin() {
			this.getIsLogin();
		},
		getIsLogin() {
			toLogin();
		},
		changeBarg(item) {
			if (!this.isLogin) {
				this.getIsLogin();
			} else {
				uni.navigateTo({
					url: `/pages/activity/goods_bargain_details/index?id=${item.id}&spid=${this.$store.state.app.uid}`
				});
			}
		},
		goDetail(item) {
			goShopDetail(item, this.$store.state.app.uid).then((res) => {
				uni.navigateTo({
					url: `/pages/goods_details/index?id=${item.id}`
				});
			});
		},
		openModal(type) {
			this.modalType = type;
			this.modalVisible = true;
			this.modalActive = false;
			if (this.modalTimer) {
				clearTimeout(this.modalTimer);
				this.modalTimer = null;
			}
			this.$nextTick(() => {
				this.modalActive = true;
			});
		},
		openModuleArticle(articleId) {
			this.openArticleModal({ id: articleId });
		},
		openArticleModal(item) {
			const articleId = typeof item === 'number' ? item : item && item.id;
			if (!articleId) {
				return;
			}
			this.modalType = 'article';
			this.articleDetail = null;
			this.articleDescription = '';
			this.articleLoading = true;
			this.modalVisible = true;
			this.modalActive = false;
			if (this.modalTimer) {
				clearTimeout(this.modalTimer);
				this.modalTimer = null;
			}
			this.$nextTick(() => {
				this.modalActive = true;
			});
			getArticleDetails(articleId)
				.then((res) => {
					this.articleDetail = res.data;
					let html = res.data && res.data.content ? res.data.content : '';
					if (html) {
						html = html.replace(/<img/gi, '<img style="max-width:100%;height:auto;float:left;display:block" ');
						html = html.replace(/<video/gi, '<video style="width:100%;height:300px;display:block" ');
					}
					this.articleDescription = html;
					this.articleLoading = false;
				})
				.catch(() => {
					this.articleLoading = false;
				});
		},
		openVirtual() {
			this.modalType = 'virtual';
			this.panoIndex = 0;
			this.panoList = [];
			for (let i = 1; i <= 8; i++) {
				this.panoList.push(`/picture720/p_${i}.jpg`);
			}
			this.modalVisible = true;
			this.modalActive = false;
			if (this.modalTimer) {
				clearTimeout(this.modalTimer);
				this.modalTimer = null;
			}
			this.$nextTick(() => {
				this.modalActive = true;
			});
		},
		closeModal() {
			this.modalActive = false;
			if (this.modalTimer) {
				clearTimeout(this.modalTimer);
				this.modalTimer = null;
			}
			this.modalTimer = setTimeout(() => {
				this.modalVisible = false;
			}, 260);
		},
		panoChange(e) {
			this.panoIndex = e.detail.current;
		},
		newDataStatus(val, num) {
			this.isFooter = val ? true : false;
			this.pdHeight = num;
		},
		// #ifdef H5
		// 微信分享；
		setOpenShare: function () {
			let that = this;
			let uid = this.uid ? this.uid : 0;
			if (that.$wechat.isWeixin()) {
				getShare().then((res) => {
					let data = res.data;
					let configAppMessage = {
						desc: data.synopsis,
						title: data.title,
						link: location.href + '?spid=' + uid,
						imgUrl: data.img
					};
					that.$wechat.wechatEvevt(['updateAppMessageShareData', 'updateTimelineShareData', 'onMenuShareAppMessage', 'onMenuShareTimeline'], configAppMessage);
				});
			}
		}
		// #endif
	},
	onReachBottom() {
		if (this.goodList.length) {
			this.getGoodsList();
		}
	},
	onPageScroll(e) {
		if (e.scrollTop > 20) {
			this.myApplet = false;
		}
		// #ifdef H5
		if (this.isHeaderSerch) {
			if (e.scrollTop > this.domOffsetTop) {
				this.isFixed = true;
			}
			if (e.scrollTop < this.domOffsetTop) {
				this.$nextTick(() => {
					this.isFixed = false;
				});
			}
		} else {
			this.isFixed = false;
		}
		// #endif
		if (e.scrollTop > 10) {
			this.isScrolled = true;
		} else {
			this.isScrolled = false;
		}
		uni.$emit('scroll');
		uni.$emit('onPageScroll', e.scrollTop);
	},
	//#ifdef MP
	onShareAppMessage() {
		let uid = this.uid ? this.uid : 0;
		if (this.shareInfo.img) {
			return {
				title: this.shareInfo.title,
				path: '/pages/index/index?spid=' + uid,
				imageUrl: this.shareInfo.img,
				desc: this.shareInfo.synopsis
			};
		} else {
			return {
				title: this.shareInfo.title,
				path: '/pages/index/index?spid=' + uid
				// imageUrl: this.shareInfo.img,
				// desc: this.shareInfo.synopsis
			};
		}
	},
	//分享到朋友圈
	onShareTimeline: function () {
		return {
			title: this.shareInfo.title,
			path: '/pages/index/index',
			imageUrl: this.shareInfo.img,
			desc: this.shareInfo.synopsis
		};
	}
	//#endif
};
</script>

<style lang="scss">
.page {
	// padding-bottom: 50px;
	overflow-y: scroll;
	overflow-x: hidden;
	position: relative;
	--panel-border: #B7DBB6;
	--panel-border-width: 1rpx;
	--panel-radius: 20rpx;
}
.myApplet {
	position: relative;
	&::after {
		position: absolute;
		right: 55px;
		top: -5px;
		content: '';
		width: 0;
		height: 0;
		border-left: 7px solid transparent;
		border-right: 7px solid transparent;
		border-bottom: 7px solid #fff;
	}
}
.pictrue_log_class {
	background-color: var(--view-theme);
}

.ysize {
	background-size: 100%;
}

.fullsize {
	background-size: 100% 100%;
}

.repeat {
	background-repeat: repeat;
}

.noRepeat {
	background-repeat: no-repeat;
}

.home-logo {
	position: absolute;
	top: 12rpx;
	left: 16rpx;
	height: auto;
	z-index: 2001;
	pointer-events: none;
}

.notice-and-entrances {
	position: relative;
	z-index: 3;
	padding: 12rpx 20rpx 0;
}

.notice-wrap {
	margin-bottom: 12rpx;
	border-radius: var(--panel-radius);
	overflow: hidden;
	border: 0.5rpx solid var(--panel-border);
	box-sizing: border-box;
	--notice-inner-radius: var(--panel-radius);
}

.entrance-row {
	display: flex;
	justify-content: space-between;
	gap: 12rpx;
	margin-top: 16rpx;
}

.entrance-item {
	flex: 1;
	background: #FFFFFF;
	border-radius: 20rpx;
	padding: 14rpx 0 12rpx;
	display: flex;
	flex-direction: column;
	align-items: center;
	box-shadow: 0 10rpx 20rpx rgba(0, 0, 0, 0.08);
	border: 1rpx solid var(--panel-border);
}

.entrance-icon-img {
	width: 64rpx;
	height: 64rpx;
	border-radius: 16rpx;
}

.entrance-text {
	margin-top: 10rpx;
	font-size: 24rpx;
	color: #000000;
}

.virtual-card {
	margin-top: 16rpx;
	height: 120rpx;
	border-radius: 28rpx;
	background-size: cover;
	background-position: center;
	position: relative;
	overflow: hidden;
	display: flex;
	align-items: center;
	padding: 0 24rpx;
	box-shadow: 0 12rpx 24rpx rgba(0, 0, 0, 0.12);
	border: 1rpx solid var(--panel-border);
}

.virtual-title {
	font-size: 30rpx;
	font-weight: 600;
	color: #ffffff;
	text-shadow: 0 6rpx 12rpx rgba(0, 0, 0, 0.3);
}

.modal-target {
	position: relative;
	border-radius: var(--panel-radius);
	overflow: hidden;
	border: 0.5rpx solid var(--panel-border);
	box-sizing: border-box;
}

.modal-cover {
	position: absolute;
	left: 0;
	top: 0;
	right: 0;
	bottom: 0;
	z-index: 6;
}

.modal-target .articleList,
.modal-target .articleList .item {
	border-radius: var(--panel-radius);
	overflow: hidden;
}

.notice-wrap {
	position: relative;
}

.notice-wrap > view:not(.modal-cover) {
	margin-top: 0 !important;
	padding: 0 !important;
	box-sizing: border-box;
}

.modal-mask {
	position: fixed;
	left: 0;
	top: 0;
	right: 0;
	bottom: 0;
	background: rgba(0, 0, 0, 0.55);
	display: flex;
	align-items: center;
	justify-content: center;
	z-index: 2005;
	opacity: 0;
	visibility: hidden;
	transition: opacity 260ms ease, visibility 260ms ease;
}

.modal-mask-show {
	opacity: 1;
	visibility: visible;
}

.modal-panel {
	width: 92%;
	height: 92vh;
	max-height: 92vh;
	background: #ffffff;
	border-radius: 32rpx;
	overflow: hidden;
	display: flex;
	flex-direction: column;
	transform: translateY(18vh);
	opacity: 0;
	transition: transform 360ms ease, opacity 360ms ease;
	border: 1rpx solid var(--panel-border);
}

.modal-panel-show {
	transform: translateY(0);
	opacity: 1;
}

.modal-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 24rpx 28rpx 18rpx;
}

.modal-title {
	font-size: 30rpx;
	font-weight: 600;
	color: #1a1a1a;
}

.modal-close {
	font-size: 36rpx;
	color: #333333;
	line-height: 1;
}

.modal-body {
	padding: 0 28rpx 28rpx;
	overflow: auto;
	flex: 1;
}

.modal-text {
	font-size: 26rpx;
	color: #333333;
	padding: 10rpx 0 30rpx;
}

.pano-swiper {
	width: 100%;
	height: 520rpx;
	border-radius: 24rpx;
	overflow: hidden;
	border: 1rpx solid var(--panel-border);
}

.pano-image {
	width: 100%;
	height: 520rpx;
}

.pano-indicator {
	text-align: center;
	margin-top: 12rpx;
	font-size: 24rpx;
	color: #666666;
}

.modal-section {
	padding: 6rpx 0 10rpx;
}

.article-detail {
	padding-bottom: 10rpx;
}

.article-title {
	font-size: 34rpx;
	color: #282828;
	font-weight: 600;
	line-height: 1.5;
}

.article-meta {
	margin-top: 18rpx;
	display: flex;
	align-items: center;
	color: #b1b2b3;
	font-size: 24rpx;
}

.article-label {
	margin-right: 16rpx;
}

.article-meta-item {
	margin-left: 16rpx;
	display: inline-flex;
	align-items: center;
}

.article-meta-item .iconfont {
	font-size: 28rpx;
	margin-right: 8rpx;
}

.article-content {
	margin-top: 20rpx;
	font-size: 28rpx;
	color: #8a8b8c;
}

.index > *:not(.pb-safe):not(.site-config):not(.site-config-placeholder):not(.modal-mask):not(pageFooter):not(.modal-target) {
	border: 1rpx solid var(--panel-border);
	border-radius: 20rpx;
	overflow: hidden;
}

.error-network {
	position: fixed;
	left: 0;
	top: 0;
	display: flex;
	flex-direction: column;
	align-items: center;
	width: 100%;
	height: 100%;
	padding-top: 40rpx;
	background: #fff;

	image {
		width: 414rpx;
		height: 336rpx;
	}

	.title {
		position: relative;
		top: -40rpx;
		font-size: 32rpx;
		color: #666;
	}

	.con {
		font-size: 24rpx;
		color: #999;

		.label {
			margin-bottom: 20rpx;
		}

		.item {
			margin-bottom: 20rpx;
		}
	}

	.btn {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 508rpx;
		height: 86rpx;
		margin-top: 100rpx;
		border: 1px solid #d74432;
		color: #e93323;
		font-size: 30rpx;
		border-radius: 120rpx;
	}
}

.sort-scroll {
	background-color: #fff;
}

.sort-product {
	margin-top: 20rpx;
}

.site-config {
	margin: 40rpx 0;
	font-size: 24rpx;
	text-align: center;
	color: #666;
	display: flex;
	align-items: center;
	justify-content: center;
	.ban {
		width: 22rpx;
		height: 24rpx;
		margin-right: 10rpx;
	}
	&.fixed {
		position: fixed;
		bottom: 69px;
		left: 0;
		width: 100%;
	}
}
.select {
	border: 1px solid var(--view-theme);
}
</style>
