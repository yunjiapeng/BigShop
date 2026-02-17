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
					<view v-for="group in courseGroups" :key="group.key" class="section">
						<view class="section-title">{{ group.title }}</view>
						<view class="course-list">
							<view v-for="item in group.items" :key="item.id" class="course-card" @click="viewCourse(item)">
								<image class="course-card-bg" :src="item.recommend_image || item.image" mode="aspectFill"></image>
								<view class="course-card-scrim"></view>
								<view class="course-card-info">
									<view class="course-card-title">{{ item.store_name }}</view>
								</view>
							</view>
						</view>
					</view>
				</scroll-view>
			</swiper-item>

			<swiper-item>
				<scroll-view class="pane" scroll-y="true">
					<view class="section">
						<view class="section-title">班级动态</view>
						<emptyPage v-if="!isLogin" :title="$t(`请先登录查看班级动态`)"></emptyPage>
						<view v-else>
							<view class="hero-card" v-if="currentCourseCard" @click="viewCourse(currentCourseCard)">
								<image
									class="hero-card-bg"
									:src="currentCourseCard.recommend_image || currentCourseCard.image || officialCover"
									mode="aspectFill"
								></image>
								<view class="hero-card-scrim"></view>
								<view class="hero-card-top">
									<view class="hero-card-title">{{ currentCourseCard.store_name || courseCategoryTitle || '当前课程' }}</view>
									<view v-if="getSubtitle(currentCourseCard.store_name)" class="hero-card-desc">
										{{ getSubtitle(currentCourseCard.store_name) }}
									</view>
								</view>
								<view class="hero-card-tabs" v-if="showClassTabs">
									<scroll-view class="hero-tab-row" scroll-x="true" @click.stop>
										<view
											v-for="cls in classList"
											:key="getClassKey(cls)"
											class="hero-tab"
											:class="{ active: isClassActive(cls) }"
											@click.stop="selectClass(cls)"
										>
											<view class="hero-tab-text">{{ getClassName(cls) }}</view>
										</view>
									</scroll-view>
								</view>
								<view class="feature-actions">
									<view v-for="b in classActionButtons" :key="b" class="feature-action" @click.stop="tapAction(b)">
										<view class="feature-action-text">{{ b }}</view>
									</view>
								</view>
							</view>
							<view class="filter-card">
								<view class="filter-label">我的孩子</view>
								<scroll-view class="chip-row" scroll-x="true">
									<view
										v-for="child in children"
										:key="child.child_uid"
										class="chip"
										:class="{ active: Number(child.child_uid) === Number(selectedChildId) }"
										@click="selectChild(child)"
									>
										<view class="chip-text">{{ child.child_name }}</view>
									</view>
									<view v-if="!childLoading && !children.length" class="chip-empty">暂无孩子</view>
								</scroll-view>
							</view>
							<view class="gallery-grid" v-if="galleryList.length">
								<view
									v-for="(item, idx) in galleryList"
									:key="item.id || idx"
									class="gallery-item"
									@click="previewGallery(idx)"
								>
									<image class="gallery-img" :src="item.pic_url" mode="aspectFill"></image>
									<view v-if="formatGalleryTime(item)" class="gallery-time">{{ formatGalleryTime(item) }}</view>
								</view>
							</view>
							<emptyPage v-else-if="showGalleryEmpty" :title="$t(`暂无班级动态`)"></emptyPage>
						</view>
					</view>
				</scroll-view>
			</swiper-item>

			<swiper-item>
				<scroll-view class="pane" scroll-y="true">
					<view v-for="group in campGroups" :key="group.key" class="section">
						<view class="section-title">{{ group.title }}</view>
						<view :class="group.layout === 'double' ? 'camp-grid' : 'camp-list'">
							<view
								v-for="item in group.items"
								:key="item.id"
								:class="['camp-card', group.layout === 'double' ? 'camp-card-compact' : '']"
								@click="viewCourse(item)"
							>
								<image class="camp-card-bg" :src="item.recommend_image || item.image || campCover" mode="aspectFill"></image>
								<view class="camp-card-scrim"></view>
								<view class="camp-card-info">
									<view class="camp-card-title">{{ item.store_name }}</view>
								</view>
							</view>
						</view>
						<emptyPage v-if="!group.items.length" :title="$t(`暂无${group.title}`)"></emptyPage>
					</view>
				</scroll-view>
			</swiper-item>
		</swiper>
		<pageFooter></pageFooter>
	</view>
</template>

<script>
import { getCategoryList, getProductslist } from '@/api/store.js';
import { getChildrenList, getMyClasses, getChildGallery } from '@/api/user.js';
import pageFooter from '@/components/pageFooter/index.vue';
import emptyPage from '@/components/emptyPage.vue';
import { mapGetters } from 'vuex';

export default {
	components: {
		pageFooter,
		emptyPage
	},
	data() {
		return {
			tabs: [
				{ key: 'select', title: '购买课程' },
				{ key: 'official', title: '班级动态' },
				{ key: 'camp', title: '主题营/周边' }
			],
			currentTab: 0,
			courseCategoryTitle: '',
			courseCid: 0,
			courseList: [],
			classList: [],
			classLoading: false,
			children: [],
			childLoading: false,
			selectedClassId: '',
			selectedChildId: '',
			galleryList: [],
			galleryPage: 1,
			galleryLimit: 200,
			galleryCount: 0,
			galleryLoading: false,
			campProducts: [],
			campCategories: [],
			campCategoryProducts: {},
			courseGroupConfig: [
				{ key: 'regular', title: '平时课程', ids: [17, 15] },
				{ key: 'winter', title: '寒假课程', ids: [10, 9] },
				{ key: 'summer', title: '暑假课程', ids: [13, 11] }
			],
			courseIdMeta: {
				17: { store_name: '平日 - 国际班' },
				15: { store_name: '平日 - 森林班' },
				13: { store_name: '暑假 - 国际班' },
				11: { store_name: '暑假 - 森林班' },
				10: { store_name: '寒假 - 森林班' },
				9: { store_name: '寒假 - 国际班' }
			},
			officialCover: '',
			campCover: '',
			actionButtons: ['课程介绍', '课程表', '作业', '资料'],
			classActionButtons: ['近期动态', '活动评价', '联系老师'],
			campWaterfall: []
		};
	},
	computed: {
		...mapGetters(['isLogin']),
		showClassTabs() {
			return Array.isArray(this.classList) && this.classList.length > 1;
		},
		currentCourseCard() {
			const list = Array.isArray(this.courseList) ? this.courseList : [];
			if (list.length) return list[0];
			const fallback = this.getMockCourses();
			return fallback.length ? fallback[0] : null;
		},
		campGroups() {
			const groups = [
				{ key: 'parent', title: '家长课堂', label: '家长课堂', layout: 'single' },
				{ key: 'family', title: '亲子活动', label: '亲子活动', layout: 'single' },
				{ key: 'peripheral', title: '周边商品', label: '周边商品', layout: 'double' }
			];
			return groups.map((group) => ({
				key: group.key,
				title: group.title,
				layout: group.layout,
				items: this.getCampGroupItems(group)
			}));
		},
		courseGroups() {
			const list = Array.isArray(this.courseList) ? this.courseList : [];
			return this.courseGroupConfig.map((group) => {
				const items = group.ids
					.map((id) => {
						const found = list.find((item) => Number(item.id) === Number(id));
						if (found) return found;
						const meta = this.courseIdMeta[id] || {};
						return {
							id,
							store_name: meta.store_name || `课程-${id}`,
							image: meta.image || this.officialCover,
							recommend_image: meta.recommend_image || this.campCover
						};
					})
					.filter(Boolean);
				return {
					key: group.key,
					title: group.title,
					items
				};
			});
		},
		campWaterfallLeft() {
			return this.campWaterfall.filter((_, idx) => idx % 2 === 0);
		},
		campWaterfallRight() {
			return this.campWaterfall.filter((_, idx) => idx % 2 === 1);
		},
		showGalleryEmpty() {
			return (
				this.isLogin &&
				!this.galleryLoading &&
				this.children.length &&
				!this.galleryList.length
			);
		}
	},
	watch: {
		isLogin: {
			handler(val) {
				if (val) this.loadClassDynamic();
			},
			deep: true
		}
	},
	async onLoad() {
		this.initUiData();
		await this.loadCategoriesAndCourses();
		await this.loadCampProducts();
		if (this.isLogin) this.loadClassDynamic();
	},
	methods: {
		initUiData() {
			const fallbackCover =
				'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/ec37d202602071627534044.jpg?imageMogr2/thumbnail/800x800';
			this.officialCover = fallbackCover;
			this.campCover = 'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/bf19c202602071647134864.jpg?imageMogr2/thumbnail/800x800';
			const fallbackList = [
				fallbackCover,
				this.campCover,
				'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/ec37d202602071627534044.jpg?imageMogr2/thumbnail/700x700',
				'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/bf19c202602071647134864.jpg?imageMogr2/thumbnail/700x700',
				'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/ec37d202602071627534044.jpg?imageMogr2/thumbnail/600x600',
				'https://saas-1375106378.cos.ap-shanghai.myqcloud.com/attach/2026/02/bf19c202602071647134864.jpg?imageMogr2/thumbnail/600x600'
			];
			this.campWaterfall = [...fallbackList].reverse();
		},
		async loadClassDynamic() {
			await Promise.all([this.loadClassList(), this.loadChildrenList()]);
			this.tryLoadGallery(true);
		},
		async loadClassList() {
			this.classLoading = true;
			try {
				const res = await getMyClasses();
				const data = (res && res.data) || {};
				const list = Array.isArray(data.list) ? data.list : Array.isArray(data) ? data : [];
				this.classList = this.dedupeClassList(list);
				if (!this.selectedClassId && list.length) {
					this.selectedClassId = this.getClassKey(this.classList[0]);
				}
			} catch (e) {
				this.classList = [];
			}
			this.classLoading = false;
		},
		async loadChildrenList() {
			this.childLoading = true;
			try {
				const res = await getChildrenList();
				this.children = (res && res.data && res.data.list) || [];
				if (!this.selectedChildId && this.children.length) {
					this.selectedChildId = this.children[0].child_uid;
				}
			} catch (e) {
				this.children = [];
			}
			this.childLoading = false;
		},
		getClassKey(item) {
			if (!item) return '';
			return item.class_id || item.classid || item.id || '';
		},
		dedupeClassList(list) {
			if (!Array.isArray(list) || !list.length) return [];
			const seen = new Set();
			const result = [];
			list.forEach((item) => {
				const key = this.getClassKey(item) || this.getClassName(item);
				if (!key || seen.has(String(key))) return;
				seen.add(String(key));
				result.push(item);
			});
			return result;
		},
		getClassName(item) {
			if (!item) return '';
			return item.class_name || item.name || item.title || `班级${this.getClassKey(item)}`;
		},
		isClassActive(item) {
			return Number(this.getClassKey(item)) === Number(this.selectedClassId);
		},
		selectClass(item) {
			const id = this.getClassKey(item);
			if (!id || Number(id) === Number(this.selectedClassId)) return;
			this.selectedClassId = id;
			this.tryLoadGallery(true);
		},
		selectChild(item) {
			if (!item || !item.child_uid || Number(item.child_uid) === Number(this.selectedChildId)) return;
			this.selectedChildId = item.child_uid;
			this.tryLoadGallery(true);
		},
		tryLoadGallery(reset) {
			if (!this.selectedChildId) {
				if (reset) this.galleryList = [];
				return;
			}
			this.loadGallery(reset);
		},
		async loadGallery(reset) {
			if (this.galleryLoading) return;
			this.galleryLoading = true;
			if (reset) this.galleryPage = 1;
			try {
				const res = await getChildGallery({
					child_uid: this.selectedChildId,
					page: this.galleryPage,
					limit: this.galleryLimit
				});
				const data = (res && res.data) || {};
				const list = Array.isArray(data.list) ? data.list : Array.isArray(data) ? data : [];
				this.galleryList = reset ? list : this.galleryList.concat(list);
				this.galleryCount = data.count || this.galleryList.length;
			} catch (e) {
				if (reset) this.galleryList = [];
			}
			this.galleryLoading = false;
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
		previewGallery(idx) {
			const urls = this.galleryList.map((item) => item.pic_url).filter(Boolean);
			if (!urls.length) return;
			uni.previewImage({
				current: urls[idx] || urls[0],
				urls
			});
		},
		async loadCategoriesAndCourses() {
			let categories = [];
			try {
				const res = await getCategoryList();
				categories = (res && res.data) || [];
				if (categories.length) {
					this.courseCategoryTitle = categories[0].cate_name || '';
					this.courseCid = categories[0].id || 0;
				}
			} catch (e) {}
			this.campCategories = categories;

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
		async loadCampProducts() {
			try {
				const res = await getProductslist({
					page: 1,
					limit: 100,
					type: 1,
					sid: 0
				});
				this.campProducts = Array.isArray(res && res.data) ? res.data : [];
			} catch (e) {
				this.campProducts = [];
			}
			const peripheralId = this.getCategoryIdByName('周边商品');
			if (peripheralId) {
				try {
					const res = await getProductslist({
						page: 1,
						limit: 100,
						type: 1,
						cate_id: peripheralId,
						sid: 0
					});
					this.$set(this.campCategoryProducts, '周边商品', Array.isArray(res && res.data) ? res.data : []);
				} catch (e) {
					this.$set(this.campCategoryProducts, '周边商品', []);
				}
			} else {
				this.$set(this.campCategoryProducts, '周边商品', []);
			}
		},
		getCategoryIdByName(name) {
			const list = Array.isArray(this.campCategories) ? this.campCategories : [];
			const match = list.find((item) => item && String(item.cate_name) === String(name));
			if (match && match.id) return match.id;
			const childMatch = list
				.map((item) => (item && Array.isArray(item.children) ? item.children : []))
				.reduce((acc, cur) => acc.concat(cur), [])
				.find((item) => item && String(item.cate_name) === String(name));
			return childMatch && childMatch.id ? childMatch.id : '';
		},
		getItemLabelNames(item) {
			const list = Array.isArray(item && item.label_list) ? item.label_list : [];
			return list
				.map((label) => label && (label.name || label.title || label.label_name))
				.filter(Boolean);
		},
		getItemCateIds(item) {
			const raw = item && (item.cate_id || item.cid || item.category_id);
			if (!raw) return [];
			if (Array.isArray(raw)) return raw.map((id) => String(id));
			return String(raw)
				.split(',')
				.map((id) => id.trim())
				.filter(Boolean);
		},
		filterCampByLabel(label) {
			const list = Array.isArray(this.campProducts) ? this.campProducts : [];
			return list.filter((item) => {
				const names = this.getItemLabelNames(item);
				return names.some((name) => String(name).includes(label));
			});
		},
		filterCampByCategoryName(name) {
			const categoryId = this.getCategoryIdByName(name);
			if (!categoryId) return [];
			const list = Array.isArray(this.campProducts) ? this.campProducts : [];
			return list.filter((item) => {
				const ids = this.getItemCateIds(item);
				return ids.some((id) => Number(id) === Number(categoryId));
			});
		},
		getCampGroupItems(group) {
			const byLabel = this.filterCampByLabel(group.label);
			if (byLabel.length) return byLabel;
			if (group.label === '周边商品') {
				const byCategory =
					this.campCategoryProducts['周边商品'] && this.campCategoryProducts['周边商品'].length
						? this.campCategoryProducts['周边商品']
						: this.filterCampByCategoryName(group.label);
				return byCategory;
			}
			return byLabel;
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
			if (!item || !item.id) return;
			uni.navigateTo({
				url: `/pages/goods_details/index?id=${item.id}`
			});
		},
		tapAction(b) {
			if (b === '活动评价') {
				const current = this.currentCourseCard || {};
				const productId = current.id || '';
				if (!productId) {
					return uni.showToast({
						title: '暂无课程信息',
						icon: 'none'
					});
				}
				const name = encodeURIComponent(current.store_name || '');
				return uni.navigateTo({
					url: `/pages/course/activity_review/index?product_id=${productId}&name=${name}`
				});
			}
			if (b === '近期动态') {
				const classId = this.selectedClassId || '';
				const childUid = this.selectedChildId || '';
				if (!classId) {
					return uni.showToast({
						title: '请选择班级',
						icon: 'none'
					});
				}
				const className = encodeURIComponent(this.getClassName(this.classList.find((item) => this.getClassKey(item) === classId)));
				return uni.navigateTo({
					url: `/pages/course/class_dynamic/index?class_id=${classId}&child_uid=${childUid}&name=${className}`
				});
			}
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
	background: #f6fbf7;
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
	background: linear-gradient(180deg, rgba(246, 251, 247, 0.98) 0%, rgba(246, 251, 247, 0.78) 55%, rgba(246, 251, 247, 0) 100%);
	pointer-events: none;
}

.tabs {
	position: relative;
	display: flex;
	align-items: center;
	justify-content: space-between;
}

.tab {
	position: relative;
	display: flex;
	align-items: center;
	justify-content: center;
	height: 78rpx;
	flex: 1;
	background: transparent;
	border: 0;
	transition: transform 180ms ease, opacity 180ms ease;
}

.tab:active {
	transform: scale(0.98);
	opacity: 0.92;
}

.tab-text {
	font-family: SemiBold;
	letter-spacing: 4rpx;
	font-size: 30rpx;
	color: rgba(17, 59, 46, 0.62);
}

.tab.active .tab-text {
	color: rgba(14, 124, 95, 0.98);
}

.tab.active::after {
	content: '';
	position: absolute;
	left: 44rpx;
	right: 44rpx;
	bottom: 14rpx;
	height: 3rpx;
	border-radius: 999rpx;
	background: linear-gradient(90deg, rgba(123, 228, 149, 0.2) 0%, rgba(86, 197, 150, 0.98) 40%, rgba(123, 228, 149, 0.2) 100%);
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
	color: rgba(17, 59, 46, 0.92);
	margin: 12rpx 0 18rpx;
}

.course-list {
	display: flex;
	flex-direction: column;
	gap: 18rpx;
}

.course-card {
	position: relative;
	height: 260rpx;
	border-radius: 24rpx;
	overflow: hidden;
	background: rgba(255, 255, 255, 0.9);
	border: 1rpx solid rgba(17, 59, 46, 0.08);
}

.course-card-bg {
	position: absolute;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;
}

.course-card-scrim {
	position: absolute;
	left: 0;
	right: 0;
	bottom: 0;
	height: 190rpx;
	background: linear-gradient(180deg, rgba(246, 251, 247, 0) 0%, rgba(246, 251, 247, 0.62) 42%, rgba(246, 251, 247, 0.92) 100%);
}

.course-card-info {
	position: absolute;
	left: 18rpx;
	right: 18rpx;
	bottom: 16rpx;
	padding: 0;
	background: transparent;
	border: 0;
	box-shadow: none;
}

.course-card-title {
	font-family: SemiBold;
	font-size: 34rpx;
	line-height: 42rpx;
	color: #000000;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.course-card:active {
	transform: scale(0.992);
}

.camp-list {
	display: flex;
	flex-direction: column;
	gap: 14rpx;
}

.camp-grid {
	display: grid;
	grid-template-columns: repeat(2, 1fr);
	gap: 14rpx;
}

.camp-card {
	position: relative;
	height: 210rpx;
	border-radius: 22rpx;
	overflow: hidden;
	background: rgba(255, 255, 255, 0.92);
	border: 1rpx solid rgba(17, 59, 46, 0.08);
}

.camp-card-compact {
	height: 190rpx;
}

.camp-card-bg {
	position: absolute;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;
}

.camp-card-scrim {
	position: absolute;
	left: 0;
	right: 0;
	bottom: 0;
	height: 150rpx;
	background: linear-gradient(180deg, rgba(246, 251, 247, 0) 0%, rgba(246, 251, 247, 0.6) 42%, rgba(246, 251, 247, 0.92) 100%);
}

.camp-card-info {
	position: absolute;
	left: 16rpx;
	right: 16rpx;
	bottom: 12rpx;
}

.camp-card-title {
	font-family: SemiBold;
	font-size: 28rpx;
	line-height: 38rpx;
	color: #000000;
	white-space: nowrap;
	overflow: hidden;
	text-overflow: ellipsis;
}

.camp-card:active {
	transform: scale(0.992);
}

.hero-card {
	position: relative;
	height: 320rpx;
	border-radius: 28rpx;
	overflow: hidden;
	background: rgba(255, 255, 255, 0.92);
	border: 1rpx solid rgba(17, 59, 46, 0.08);
}

.hero-card-bg {
	position: absolute;
	left: 0;
	top: 0;
	width: 100%;
	height: 100%;
}

.hero-card-scrim {
	position: absolute;
	left: 0;
	right: 0;
	top: 0;
	bottom: 0;
	background: linear-gradient(180deg, rgba(246, 251, 247, 0.62) 0%, rgba(246, 251, 247, 0.15) 40%, rgba(246, 251, 247, 0.92) 100%);
}

.hero-card-top {
	position: absolute;
	left: 18rpx;
	right: 18rpx;
	top: 18rpx;
	padding: 0;
	background: transparent;
	border: 0;
	box-shadow: none;
}

.hero-card-title {
	font-family: SemiBold;
	font-size: 40rpx;
	letter-spacing: 2rpx;
	color: rgba(17, 59, 46, 0.94);
	text-shadow: 0 8rpx 18rpx rgba(246, 251, 247, 0.9);
}

.hero-card-desc {
	font-family: Regular;
	font-size: 26rpx;
	line-height: 36rpx;
	color: rgba(17, 59, 46, 0.7);
	text-shadow: 0 8rpx 18rpx rgba(246, 251, 247, 0.85);
}

.hero-card-tabs {
	position: absolute;
	left: 18rpx;
	right: 18rpx;
	bottom: 128rpx;
}

.hero-tab-row {
	display: flex;
	gap: 12rpx;
	white-space: nowrap;
}

.hero-tab {
	padding: 0 20rpx;
	height: 64rpx;
	border-radius: 999rpx;
	background: rgba(255, 255, 255, 0.86);
	border: 1rpx solid rgba(17, 59, 46, 0.18);
	display: inline-flex;
	align-items: center;
	justify-content: center;
}

.hero-tab.active {
	background: rgba(86, 197, 150, 0.2);
	border-color: rgba(86, 197, 150, 0.6);
}

.hero-tab-text {
	font-size: 24rpx;
	letter-spacing: 1rpx;
	color: rgba(17, 59, 46, 0.85);
}

.hero-card-bottom {
	position: absolute;
	left: 18rpx;
	right: 18rpx;
	bottom: 18rpx;
	display: flex;
	gap: 12rpx;
}

.hero-chip {
	flex: 1;
	height: 68rpx;
	border-radius: 18rpx;
	background: rgba(255, 255, 255, 0.8);
	border: 1rpx solid rgba(17, 59, 46, 0.1);
	display: flex;
	align-items: center;
	justify-content: center;
	font-size: 24rpx;
	letter-spacing: 2rpx;
	color: rgba(17, 59, 46, 0.78);
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
	background: rgba(255, 255, 255, 0.86);
	border: 1rpx solid rgba(17, 59, 46, 0.1);
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
	color: rgba(17, 59, 46, 0.78);
}

.feature-actions {
	display: grid;
	grid-template-columns: repeat(3, 1fr);
	gap: 14rpx;
	position: absolute;
	left: 18rpx;
	right: 18rpx;
	bottom: 18rpx;
}

.feature-action {
	height: 90rpx;
	border-radius: 20rpx;
	background: rgba(255, 255, 255, 0.86);
	border: 1rpx solid rgba(17, 59, 46, 0.1);
	display: flex;
	align-items: center;
	justify-content: center;
}

.feature-action:active {
	transform: scale(0.98);
}

.feature-action-text {
	font-family: Regular;
	font-size: 24rpx;
	letter-spacing: 2rpx;
	color: rgba(17, 59, 46, 0.78);
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
	background: rgba(255, 255, 255, 0.92);
	border: 1rpx solid rgba(17, 59, 46, 0.08);
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

.filter-card {
	padding: 18rpx;
	border-radius: 22rpx;
	background: rgba(255, 255, 255, 0.9);
	border: 1rpx solid rgba(17, 59, 46, 0.08);
}

.filter-label {
	font-family: SemiBold;
	font-size: 26rpx;
	letter-spacing: 1rpx;
	color: rgba(17, 59, 46, 0.78);
	margin: 6rpx 0 12rpx;
}

.chip-row {
	display: flex;
	gap: 12rpx;
	white-space: nowrap;
	margin-bottom: 18rpx;
}

.chip {
	padding: 0 20rpx;
	height: 64rpx;
	border-radius: 999rpx;
	background: rgba(246, 251, 247, 0.9);
	border: 1rpx solid rgba(17, 59, 46, 0.12);
	display: inline-flex;
	align-items: center;
	justify-content: center;
}

.chip.active {
	background: rgba(86, 197, 150, 0.16);
	border-color: rgba(86, 197, 150, 0.5);
}

.chip-text {
	font-size: 24rpx;
	letter-spacing: 1rpx;
	color: rgba(17, 59, 46, 0.78);
}

.chip-empty {
	font-size: 24rpx;
	color: rgba(17, 59, 46, 0.5);
	padding: 8rpx 0;
}

.gallery-grid {
	margin-top: 18rpx;
	display: grid;
	grid-template-columns: repeat(2, 1fr);
	gap: 16rpx;
}

.gallery-item {
	position: relative;
	border-radius: 20rpx;
	overflow: hidden;
	background: rgba(255, 255, 255, 0.9);
	border: 1rpx solid rgba(17, 59, 46, 0.08);
}

.gallery-img {
	width: 100%;
	height: 320rpx;
}

.gallery-time {
	position: absolute;
	left: 0;
	right: 0;
	bottom: 0;
	padding: 12rpx 16rpx;
	font-size: 22rpx;
	color: #ffffff;
	background: linear-gradient(180deg, rgba(0, 0, 0, 0) 0%, rgba(0, 0, 0, 0.45) 100%);
}
</style>
