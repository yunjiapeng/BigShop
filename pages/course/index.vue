<template>
	<view class="course-page">
		<view class="top-nav">
			<view class="tabs">
				<view
					v-for="(t, i) in tabs"
					:key="t.key"
					class="tab"
					:class="{ active: currentTab === i }"
					@click="tapTab(i)"
				>
					<text class="tab-text">{{ t.title }}</text>
				</view>
			</view>
		</view>

		<swiper class="content-swiper" :current="currentTab" @change="onSwiperChange">
			<swiper-item>
				<scroll-view class="pane" scroll-y="true">
					<view class="section">
						<view class="section-title">{{ courseCategoryTitle || '课程分类' }}</view>
						<view class="course-list">
							<view v-for="item in courseList" :key="item.id" class="course-card" @click="viewCourse(item)">
								<image class="course-card-bg" :src="item.recommend_image || item.image" mode="aspectFill"></image>
								<view class="course-card-mask"></view>
								<view class="course-card-content">
									<view class="course-card-title">{{ getMainTitle(item.store_name) }}</view>
									<view class="course-card-subtitle">{{ getSubtitle(item.store_name) }}</view>
								</view>
								<view class="course-card-action">
									<view class="course-card-btn">查看课程</view>
								</view>
							</view>
						</view>
					</view>
				</scroll-view>
			</swiper-item>

			<swiper-item>
				<scroll-view class="pane" scroll-y="true">
					<view class="section">
						<view class="big-card">
							<image class="big-card-bg" :src="officialCover" mode="aspectFill"></image>
							<view class="big-card-mask"></view>
							<view class="big-card-title">正式课程</view>
						</view>

						<view class="quick-actions">
							<view v-for="b in actionButtons" :key="b" class="quick-action" @click="tapAction(b)">
								<view class="quick-action-text">{{ b }}</view>
							</view>
						</view>

						<view class="waterfall">
							<view class="wf-col">
								<view v-for="(img, idx) in officialWaterfallLeft" :key="'l-' + idx" class="wf-item">
									<image class="wf-img" :src="img" mode="aspectFill"></image>
								</view>
							</view>
							<view class="wf-col">
								<view v-for="(img, idx) in officialWaterfallRight" :key="'r-' + idx" class="wf-item">
									<image class="wf-img" :src="img" mode="aspectFill"></image>
								</view>
							</view>
						</view>
					</view>
				</scroll-view>
			</swiper-item>

			<swiper-item>
				<scroll-view class="pane" scroll-y="true">
					<view class="section">
						<view class="big-card">
							<image class="big-card-bg" :src="campCover" mode="aspectFill"></image>
							<view class="big-card-mask"></view>
							<view class="big-card-title">主题营</view>
						</view>

						<view class="quick-actions">
							<view v-for="b in actionButtons" :key="b" class="quick-action" @click="tapAction(b)">
								<view class="quick-action-text">{{ b }}</view>
							</view>
						</view>

						<view class="waterfall">
							<view class="wf-col">
								<view v-for="(img, idx) in campWaterfallLeft" :key="'cl-' + idx" class="wf-item">
									<image class="wf-img" :src="img" mode="aspectFill"></image>
								</view>
							</view>
							<view class="wf-col">
								<view v-for="(img, idx) in campWaterfallRight" :key="'cr-' + idx" class="wf-item">
									<image class="wf-img" :src="img" mode="aspectFill"></image>
								</view>
							</view>
						</view>
					</view>
				</scroll-view>
			</swiper-item>
		</swiper>
		<pageFooter></pageFooter>
	</view>
</template>

<script>
import { getCategoryList, getProductslist } from '@/api/store.js';
import pageFooter from '@/components/pageFooter/index.vue';

export default {
	components: {
		pageFooter
	},
	data() {
		return {
			tabs: [
				{ key: 'select', title: '选课' },
				{ key: 'official', title: '正式课程' },
				{ key: 'camp', title: '主题营' }
			],
			currentTab: 0,
			courseCategoryTitle: '',
			courseCid: 0,
			courseList: [],
			officialCover: '',
			campCover: '',
			actionButtons: ['课程介绍', '课程表', '作业', '资料'],
			officialWaterfall: [],
			campWaterfall: []
		};
	},
	computed: {
		officialWaterfallLeft() {
			return this.officialWaterfall.filter((_, idx) => idx % 2 === 0);
		},
		officialWaterfallRight() {
			return this.officialWaterfall.filter((_, idx) => idx % 2 === 1);
		},
		campWaterfallLeft() {
			return this.campWaterfall.filter((_, idx) => idx % 2 === 0);
		},
		campWaterfallRight() {
			return this.campWaterfall.filter((_, idx) => idx % 2 === 1);
		}
	},
	onLoad() {
		this.initUiData();
		this.loadCategoriesAndCourses();
	},
	methods: {
		initUiData() {
			const fallbackCover =
				'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/ec37d202602071627534044.jpg?imageMogr2/thumbnail/800x800';
			this.officialCover = fallbackCover;
			this.campCover = 'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/bf19c202602071647134864.jpg?imageMogr2/thumbnail/800x800';
			this.officialWaterfall = [
				fallbackCover,
				this.campCover,
				'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/ec37d202602071627534044.jpg?imageMogr2/thumbnail/700x700',
				'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/bf19c202602071647134864.jpg?imageMogr2/thumbnail/700x700',
				'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/ec37d202602071627534044.jpg?imageMogr2/thumbnail/600x600',
				'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/bf19c202602071647134864.jpg?imageMogr2/thumbnail/600x600'
			];
			this.campWaterfall = [...this.officialWaterfall].reverse();
		},
		async loadCategoriesAndCourses() {
			try {
				const res = await getCategoryList();
				const list = (res && res.data) || [];
				if (list.length) {
					this.courseCategoryTitle = list[0].cate_name || '';
					this.courseCid = list[0].id || 0;
				}
			} catch (e) {}

			if (!this.courseCid) {
				this.courseCategoryTitle = this.courseCategoryTitle || '课程分类';
				this.courseList = this.getMockCourses();
				return;
			}

			try {
				const res = await getProductslist({
					page: 1,
					limit: 10,
					type: 1,
					cid: this.courseCid,
					sid: 0
				});
				this.courseList = (res && res.data) || [];
				if (!this.courseList.length) this.courseList = this.getMockCourses();
			} catch (e) {
				this.courseList = this.getMockCourses();
			}
		},
		getMockCourses() {
			return [
				{
					id: 6,
					store_name: '课程分类-陈千语',
					cate_id: '1',
					image: 'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/ec37d202602071627534044.jpg?imageMogr2/thumbnail/800x800',
					recommend_image: 'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/bf19c202602071647134864.jpg?imageMogr2/thumbnail/800x800'
				},
				{
					id: 7,
					store_name: '课程分类-基础班',
					cate_id: '1',
					image: 'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/bf19c202602071647134864.jpg?imageMogr2/thumbnail/800x800',
					recommend_image: 'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/ec37d202602071627534044.jpg?imageMogr2/thumbnail/800x800'
				},
				{
					id: 8,
					store_name: '课程分类-进阶班',
					cate_id: '1',
					image: 'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/ec37d202602071627534044.jpg?imageMogr2/thumbnail/800x800',
					recommend_image: 'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/bf19c202602071647134864.jpg?imageMogr2/thumbnail/800x800'
				}
			];
		},
		getSubtitle(storeName) {
			const raw = String(storeName || '');
			const parts = raw.split('-');
			if (parts.length <= 1) return '';
			return parts.slice(1).join('-');
		},
		getMainTitle(storeName) {
			const raw = String(storeName || '');
			const parts = raw.split('-');
			return (parts[0] || '').trim() || raw;
		},
		tapTab(index) {
			this.currentTab = index;
		},
		onSwiperChange(e) {
			const cur = (e && e.detail && typeof e.detail.current === 'number' && e.detail.current) || 0;
			this.currentTab = cur;
		},
		viewCourse(item) {
			uni.showToast({
				title: '功能暂未开放',
				icon: 'none'
			});
		},
		tapAction(b) {
			uni.showToast({
				title: '功能暂未开放',
				icon: 'none'
			});
		}
	}
};
</script>

<style lang="scss" scoped>
.course-page {
	min-height: 100vh;
	background: #0e0e10;
}

.top-nav {
	position: sticky;
	top: 0;
	z-index: 20;
	padding: 22rpx 28rpx 12rpx;
}

.top-nav::before {
	content: '';
	position: absolute;
	left: 0;
	right: 0;
	top: 0;
	height: 170rpx;
	background: linear-gradient(180deg, rgba(14, 14, 16, 0.92) 0%, rgba(14, 14, 16, 0.42) 55%, rgba(14, 14, 16, 0) 100%);
	pointer-events: none;
}

.tabs {
	position: relative;
	display: grid;
	grid-template-columns: repeat(3, 1fr);
	gap: 12rpx;
}

.tab {
	position: relative;
	display: flex;
	align-items: center;
	justify-content: center;
	height: 78rpx;
	border-radius: 18rpx;
	background: rgba(255, 255, 255, 0.08);
	border: 1rpx solid rgba(255, 255, 255, 0.12);
	transition: transform 180ms ease, background 180ms ease, border-color 180ms ease;
}

.tab:active {
	transform: scale(0.98);
}

.tab-text {
	font-family: SemiBold;
	letter-spacing: 6rpx;
	font-size: 30rpx;
	color: rgba(255, 255, 255, 0.74);
}

.tab.active {
	background: rgba(255, 255, 255, 0.14);
	border-color: rgba(255, 255, 255, 0.18);
}

.tab.active .tab-text {
	color: rgba(255, 255, 255, 0.95);
}

.tab.active::after {
	content: '';
	position: absolute;
	left: 28rpx;
	right: 28rpx;
	bottom: 16rpx;
	height: 4rpx;
	border-radius: 2rpx;
	background: rgba(255, 255, 255, 0.92);
}

.content-swiper {
	height: calc(100vh - 112rpx - 170rpx);
}

.pane {
	height: 100%;
}

.section {
	padding: 14rpx 28rpx 40rpx;
}

.section-title {
	font-family: SemiBold;
	font-size: 36rpx;
	letter-spacing: 2rpx;
	color: rgba(255, 255, 255, 0.92);
	margin: 12rpx 0 18rpx;
}

.course-list {
	display: flex;
	flex-direction: column;
	gap: 18rpx;
}

.course-card {
	position: relative;
	height: 210rpx;
	border-radius: 24rpx;
	overflow: hidden;
	background: rgba(255, 255, 255, 0.06);
}

.course-card-bg {
	position: absolute;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;
}

.course-card-mask {
	position: absolute;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;
	background: rgba(0, 0, 0, 0.3);
}

.course-card-content {
	position: absolute;
	left: 18rpx;
	top: 18rpx;
	right: 160rpx;
}

.course-card-title {
	font-family: SemiBold;
	font-size: 34rpx;
	line-height: 42rpx;
	color: rgba(255, 255, 255, 0.96);
}

.course-card-subtitle {
	margin-top: 10rpx;
	font-family: Regular;
	font-size: 24rpx;
	line-height: 30rpx;
	color: rgba(255, 255, 255, 0.78);
}

.course-card-action {
	position: absolute;
	right: 18rpx;
	bottom: 18rpx;
}

.course-card-btn {
	padding: 14rpx 20rpx;
	border-radius: 16rpx;
	background: rgba(255, 255, 255, 0.16);
	border: 1rpx solid rgba(255, 255, 255, 0.22);
	color: rgba(255, 255, 255, 0.92);
	font-size: 24rpx;
	letter-spacing: 2rpx;
}

.big-card {
	position: relative;
	height: 320rpx;
	border-radius: 28rpx;
	overflow: hidden;
}

.big-card-bg {
	position: absolute;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;
}

.big-card-mask {
	position: absolute;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;
	background: rgba(0, 0, 0, 0.32);
}

.big-card-title {
	position: absolute;
	left: 22rpx;
	top: 22rpx;
	font-family: SemiBold;
	font-size: 40rpx;
	letter-spacing: 2rpx;
	color: rgba(255, 255, 255, 0.96);
}

.quick-actions {
	margin-top: 18rpx;
	display: grid;
	grid-template-columns: repeat(4, 1fr);
	gap: 14rpx;
}

.quick-action {
	height: 92rpx;
	border-radius: 20rpx;
	background: rgba(255, 255, 255, 0.08);
	border: 1rpx solid rgba(255, 255, 255, 0.12);
	display: flex;
	align-items: center;
	justify-content: center;
}

.quick-action:active {
	transform: scale(0.98);
}

.quick-action-text {
	font-family: Regular;
	font-size: 24rpx;
	letter-spacing: 2rpx;
	color: rgba(255, 255, 255, 0.86);
}

.waterfall {
	margin-top: 22rpx;
	display: flex;
	gap: 16rpx;
}

.wf-col {
	flex: 1;
	display: flex;
	flex-direction: column;
	gap: 16rpx;
}

.wf-item {
	border-radius: 22rpx;
	overflow: hidden;
	background: rgba(255, 255, 255, 0.06);
}

.wf-img {
	width: 100%;
	height: 360rpx;
}

.wf-col .wf-item:nth-child(2n) .wf-img {
	height: 420rpx;
}

.wf-col .wf-item:nth-child(3n) .wf-img {
	height: 300rpx;
}
</style>
