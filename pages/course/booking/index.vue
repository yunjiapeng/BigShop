<template>
	<view class="course-booking">
		<view class="top-bar">
			<view class="back-btn" @click="goBack">
				<text class="iconfont icon-xiangzuo"></text>
				<text class="back-text">返回</text>
			</view>
		</view>
		<view v-if="isHoliday" class="section">
			<view class="section-title">{{ durationTitle }}</view>
			<view class="duration-card">
				<view
					class="duration-track"
					@touchstart.stop="handleTrackStart"
					@touchmove.stop.prevent="handleTouchMove"
					@touchend="handleTouchEnd"
					@mousedown.stop.prevent="handleTrackMouseDown"
				>
					<view
						class="duration-range"
						:style="rangeStyle"
						@touchstart.stop="handleTrackStart"
						@touchmove.stop.prevent="handleTouchMove"
						@touchend="handleTouchEnd"
						@mousedown.stop.prevent="handleTrackMouseDown"
					></view>
					<view
						class="duration-handle start"
						:style="startStyle"
						@touchstart.stop="handleTouchStart('start')"
						@touchmove.stop.prevent="handleTouchMove"
						@touchend="handleTouchEnd"
						@mousedown.stop.prevent="handleMouseDown('start')"
					></view>
					<view
						class="duration-handle end"
						:style="endStyle"
						v-if="!isHolidayWeek"
						@touchstart.stop="handleTouchStart('end')"
						@touchmove.stop.prevent="handleTouchMove"
						@touchend="handleTouchEnd"
						@mousedown.stop.prevent="handleMouseDown('end')"
					></view>
					<view class="duration-nodes">
						<view
							class="duration-node"
							v-for="idx in totalSlots"
							:key="idx"
							:style="{ left: nodeLeft(idx - 1) + 'px' }"
						></view>
					</view>
				</view>
				<view class="selection-info">已选择：{{ selectionText }}</view>
			</view>
		</view>

		<view class="section">
			<view class="section-title">学员信息</view>
			<view class="form-card">
				<view class="child-select">
					<view class="child-header">
						<view class="label">选择孩子（可多选）</view>
						<view class="child-hint">已选 {{ selectedCount }} 位</view>
					</view>
					<view class="child-list">
						<view
							v-for="child in children"
							:key="child.child_uid"
							class="child-card"
							:class="{ active: isChildSelected(child) }"
							@click="toggleChildSelect(child)"
						>
							<view class="child-info">
								<view class="child-name">{{ child.child_name }}</view>
								<view class="child-meta">{{ formatChildMeta(child) }}</view>
							</view>
							<view class="child-actions-inline">
								<view v-if="isChildSelected(child)" class="child-tag">已选</view>
								<view class="child-mini-btn" @click.stop="startEditChild(child)">编辑</view>
								<view class="child-mini-btn danger" @click.stop="removeChild(child)">删除</view>
							</view>
						</view>
						<view class="child-card add-card" @click="toggleAddChild">
							<view class="add-plus">+</view>
							<view class="add-text">添加新的孩子</view>
						</view>
					</view>
					<view class="child-guide">可同时选择多位学员，系统将按人数购买</view>
				</view>
				<view v-if="!isHoliday" class="form-item">
					<view class="label">开课时间</view>
					<picker mode="date" :value="courseStartDate" @change="onCourseStartDateChange">
						<view class="picker-value">{{ courseStartDate || '请选择开课时间' }}</view>
					</picker>
				</view>
				<view v-if="showAddChild" class="form-item">
					<view class="label">孩子姓名</view>
					<input v-model="childName" type="text" placeholder="请输入孩子姓名" placeholder-class="placeholder" />
				</view>
				<view v-if="showAddChild" class="form-item">
					<view class="label">出生日期</view>
					<picker mode="date" :value="childBirthday" @change="onBirthdayChange">
						<view class="picker-value">{{ childBirthday || '请选择出生日期' }}</view>
					</picker>
				</view>
				<view v-if="showAddChild" class="form-item">
					<view class="label">性别</view>
					<radio-group class="gender-group" @change="onGenderChange">
						<label class="gender-item">
							<radio value="male" :checked="childGender === 'male'" />
							<text>男</text>
						</label>
						<label class="gender-item">
							<radio value="female" :checked="childGender === 'female'" />
							<text>女</text>
						</label>
					</radio-group>
				</view>
				<view v-if="showAddChild" class="form-item">
					<view class="label">家长电话</view>
					<input v-model="parentPhone" type="number" placeholder="请输入家长电话" placeholder-class="placeholder" />
				</view>
				<view v-if="showAddChild" class="child-save-bar">
					<view class="child-save-btn" @click="saveChild">{{ editingChildId ? '保存修改' : '保存孩子' }}</view>
					<view class="child-cancel-btn" @click="cancelAddChild">取消</view>
				</view>
			</view>
		</view>
		<view class="buy-bar">
			<view class="buy-button" @click="submitOrder">
				<view class="buy-meta">
					<text class="buy-text">购买课程</text>
					<text class="buy-count">x{{ displayCount }}</text>
				</view>
				<text class="buy-price">￥{{ totalPriceText }}</text>
			</view>
		</view>
	</view>
</template>

<script>
import { getProductDetail, postCartAdd } from '@/api/store.js';
import { getChildrenList, createChild, updateChild, deleteChild, getCourseTermMap } from '@/api/user.js';
export default {
	data() {
		return {
			productId: '',
			priceProductId: 6,
			selectedPrice: '',
			selectedSpec: '',
			selectedAttr: '',
			productInfo: {},
			attrUnique: '',
			weeks: [
				{ month: 8, week: 1 },
				{ month: 8, week: 2 },
				{ month: 8, week: 3 },
				{ month: 8, week: 4 },
				{ month: 9, week: 1 },
				{ month: 9, week: 2 },
				{ month: 9, week: 3 },
				{ month: 9, week: 4 }
			],
			startIndex: 0,
			endIndex: 3,
			trackLeft: 0,
			trackWidth: 0,
			stepWidth: 0,
			activeHandle: 'start',
			isMouseDown: false,
			specType: 'week',
			courseStartDate: '',
			childName: '',
			childBirthday: '',
			childGender: 'male',
			parentPhone: '',
			showAddChild: false,
			children: [],
			selectedChildIds: [],
			editingChildId: '',
			termDateMap: {},
			priceMap: {
				week: '',
				month: '',
				twoMonth: ''
			},
			priceLoading: false
		};
	},
	computed: {
		courseType() {
			const id = Number(this.productId);
			if ([13, 11].includes(id)) return 'summer';
			if ([10, 9].includes(id)) return 'winter';
			return 'regular';
		},
		isHoliday() {
			return this.courseType === 'summer' || this.courseType === 'winter';
		},
		holidayTitle() {
			return this.courseType === 'summer' ? '暑假' : '寒假';
		},
		isHolidayWeek() {
			return this.isHoliday && this.specType === 'week';
		},
		isHolidayMonth() {
			return this.isHoliday && this.specType === 'month';
		},
		totalSlots() {
			if (this.courseType === 'winter') return 12;
			if (this.courseType === 'summer') return 9;
			return 8;
		},
		durationTitle() {
			return this.isHoliday ? '课程时长（期）' : '课程时长';
		},
		selectedCount() {
			return this.selectedChildIds.length;
		},
		displayCount() {
			return this.selectedChildIds.length || 1;
		},
		selectedWeeks() {
			return this.endIndex - this.startIndex + 1;
		},
		slotLabels() {
			if (this.isHoliday) {
				return Array.from({ length: this.totalSlots }, (_, idx) => `第${idx + 1}期`);
			}
			return this.weeks.map((item) => `${item.month}月第${item.week}周`);
		},
		startLabel() {
			return this.slotLabels[this.startIndex] || '';
		},
		endLabel() {
			return this.slotLabels[this.endIndex] || '';
		},
		endMonthLabel() {
			const item = this.weeks[this.endIndex] || {};
			return `${item.month}月`;
		},
		selectionText() {
			if (this.isHolidayWeek) {
				return `${this.holidayTitle}课程第${this.startIndex + 1}期`;
			}
			if (this.isHolidayMonth) {
				return `${this.holidayTitle}课程第${this.startIndex + 1}期-第${this.endIndex + 1}期`;
			}
			if (this.selectedWeeks === 1) {
				return this.startLabel;
			}
			if (this.selectedWeeks === 4) {
				return this.startIndex <= 3 ? '8月' : '9月';
			}
			return '8月和9月';
		},
		currentPrice() {
			if (this.selectedPrice) return this.selectedPrice;
			if (this.selectedWeeks === 1) return this.priceMap.week;
			if (this.selectedWeeks === 4) return this.priceMap.month;
			return this.priceMap.twoMonth;
		},
		currentPriceText() {
			return this.currentPrice || '--';
		},
		totalPriceText() {
			const price = parseFloat(this.currentPrice);
			if (!isNaN(price) && this.selectedChildIds.length > 1) {
				return (price * this.selectedChildIds.length).toFixed(2);
			}
			return this.currentPriceText;
		},
		startStyle() {
			return {
				left: `${this.startLeft}px`
			};
		},
		endStyle() {
			return {
				left: `${this.endLeft}px`
			};
		},
		rangeStyle() {
			return {
				left: `${this.startLeft}px`,
				width: `${this.endLeft - this.startLeft}px`
			};
		},
		startLeft() {
			return this.stepWidth * this.startIndex;
		},
		endLeft() {
			return this.stepWidth * (this.endIndex + 1);
		}
	},
	onLoad(options) {
		this.productId = (options && options.id) || '';
		this.priceProductId = this.productId || 6;
		this.selectedPrice = (options && options.price) || '';
		this.selectedSpec = (options && options.spec) || '';
		const attrValue = (options && options.attr) || '';
			this.selectedAttr = attrValue;
		this.specType = this.getSpecType(this.selectedSpec || attrValue);
		this.resetRange();
		if (!this.selectedPrice) {
			this.loadProductPrices();
		}
		if (this.productId) {
			this.loadProductInfo();
			this.loadTermMap();
		}
		this.loadChildren();
	},
	onReady() {
		this.$nextTick(() => {
			this.initTrack();
		});
	},
	beforeDestroy() {
		this.removeMouseListeners();
	},
	methods: {
		goBack() {
			uni.navigateBack({
				delta: 1
			});
		},
		initTrack() {
			const query = uni.createSelectorQuery().in(this);
			query.select('.duration-track').boundingClientRect((rect) => {
				if (!rect) return;
				this.trackLeft = rect.left;
				this.trackWidth = rect.width;
				this.stepWidth = rect.width / this.totalSlots;
			}).exec();
		},
		getSpecType(value) {
			const text = String(value || '');
			if (text.includes('month')) return 'month';
			if (text.includes('week')) return 'week';
			if (text.includes('月')) return 'month';
			if (text.includes('周')) return 'week';
			return 'week';
		},
		resetRange() {
			if (this.isHolidayWeek) {
				this.endIndex = this.startIndex;
			}
			if (this.isHolidayMonth) {
				if (this.startIndex > this.totalSlots - 4) {
					this.startIndex = Math.max(0, this.totalSlots - 4);
				}
				this.endIndex = Math.min(this.startIndex + 3, this.totalSlots - 1);
			}
			if (this.endIndex >= this.totalSlots) {
				this.endIndex = Math.min(this.totalSlots - 1, Math.max(0, this.endIndex));
			}
			if (this.startIndex >= this.totalSlots) {
				this.startIndex = Math.max(0, this.totalSlots - 1);
			}
			if (this.startIndex > this.endIndex) {
				this.startIndex = Math.max(0, this.endIndex - 1);
			}
		},
		handleTouchStart(type) {
			this.activeHandle = type;
		},
		handleTrackStart(e) {
			if (!this.trackWidth) {
				this.initTrack();
			}
			if (!e.touches || !e.touches.length) return;
			const x = e.touches[0].pageX - this.trackLeft;
			const startDistance = Math.abs(x - this.startLeft);
			const endDistance = Math.abs(x - this.endLeft);
			this.activeHandle = startDistance <= endDistance ? 'start' : 'end';
			this.updateByClientX(e.touches[0].pageX);
		},
		handleTouchMove(e) {
			if (!this.trackWidth) {
				this.initTrack();
				return;
			}
			if (!e.touches || !e.touches.length) return;
			this.updateByClientX(e.touches[0].pageX);
		},
		handleTouchEnd() {
			this.applySnap();
		},
		handleMouseDown(type) {
			if (!this.trackWidth) {
				this.initTrack();
			}
			this.activeHandle = type;
			this.isMouseDown = true;
			this.addMouseListeners();
		},
		handleTrackMouseDown(e) {
			if (!this.trackWidth) {
				this.initTrack();
			}
			const x = e.clientX - this.trackLeft;
			const startDistance = Math.abs(x - this.startLeft);
			const endDistance = Math.abs(x - this.endLeft);
			this.activeHandle = startDistance <= endDistance ? 'start' : 'end';
			this.isMouseDown = true;
			this.addMouseListeners();
			this.updateByClientX(e.clientX);
		},
		handleMouseMove(e) {
			if (!this.isMouseDown) return;
			this.updateByClientX(e.clientX);
		},
		handleMouseUp() {
			if (!this.isMouseDown) return;
			this.isMouseDown = false;
			this.removeMouseListeners();
			this.applySnap();
		},
		nodeLeft(index) {
			return this.stepWidth * index;
		},
		addMouseListeners() {
			if (typeof window === 'undefined') return;
			window.addEventListener('mousemove', this.handleMouseMove);
			window.addEventListener('mouseup', this.handleMouseUp);
		},
		removeMouseListeners() {
			if (typeof window === 'undefined') return;
			window.removeEventListener('mousemove', this.handleMouseMove);
			window.removeEventListener('mouseup', this.handleMouseUp);
		},
		async loadChildren() {
			try {
				const res = await getChildrenList();
				this.children = (res && res.data && res.data.list) || [];
			} catch (e) {
				this.children = [];
			}
		},
		async loadTermMap() {
			if (!this.productId || !this.isHoliday) {
				this.termDateMap = {};
				return;
			}
			try {
				const res = await getCourseTermMap({
					product_id: this.productId,
					course_type: this.courseType
				});
				this.termDateMap = this.normalizeTermMap(res && res.data);
			} catch (e) {
				this.termDateMap = {};
			}
		},
		normalizeTermMap(data) {
			if (!data) return {};
			const map = {};
			if (Array.isArray(data)) {
				data.forEach((item) => {
					if (!item) return;
					const term = Number(item.term || item.period || item.index || item.no);
					if (!term) return;
					map[term] = {
						startdate: item.startdate || item.start_date || item.start,
						deadline: item.deadline || item.enddate || item.end_date || item.end
					};
				});
				return map;
			}
			if (typeof data === 'object') {
				Object.keys(data).forEach((key) => {
					const item = data[key];
					if (!item) return;
					const term = Number(key);
					if (!term) return;
					if (typeof item === 'object') {
						map[term] = {
							startdate: item.startdate || item.start_date || item.start,
							deadline: item.deadline || item.enddate || item.end_date || item.end
						};
					}
				});
			}
			return map;
		},
		async loadProductInfo() {
			try {
				const res = await getProductDetail(this.productId);
				this.productInfo = (res && res.data) || {};
				this.attrUnique = this.resolveUniqueId(this.selectedAttr, this.productInfo.productValue);
			} catch (e) {
				this.productInfo = {};
				this.attrUnique = '';
			}
		},
		resolveUniqueId(value, productValue) {
			const target = String(value || '').replace(/\s/g, '');
			if (!target || !productValue) return '';
			const values = Array.isArray(productValue) ? productValue : Object.values(productValue || {});
			for (let i = 0; i < values.length; i += 1) {
				const item = values[i] || {};
				const suk = String(item.suk || '').replace(/\s/g, '');
				if (!suk) continue;
				if (suk.split(',').some((part) => part === target) || suk.includes(target)) {
					return item.unique || '';
				}
			}
			return '';
		},
		toggleAddChild() {
			this.showAddChild = true;
			this.editingChildId = '';
			this.childName = '';
			this.childBirthday = '';
			this.childGender = 'male';
			this.parentPhone = '';
		},
		cancelAddChild() {
			this.showAddChild = false;
			this.editingChildId = '';
		},
		startEditChild(child) {
			if (!child) return;
			this.showAddChild = true;
			this.editingChildId = child.child_uid || '';
			this.childName = child.child_name || '';
			this.childBirthday = child.birth_date || '';
			this.childGender = this.normalizeGender(child.gender);
			this.parentPhone = '';
		},
		normalizeGender(gender) {
			if (gender === 'male' || gender === 'female') return gender;
			if (gender === 1 || gender === '1') return 'male';
			if (gender === 2 || gender === '2') return 'female';
			return 'male';
		},
		async saveChild() {
			if (!this.childName) {
				uni.showToast({
					title: '请填写孩子姓名',
					icon: 'none'
				});
				return;
			}
			try {
				const genderValue = this.normalizeGender(this.childGender) === 'female' ? 2 : 1;
				const payload = {
					child_name: this.childName,
					birth_date: this.childBirthday,
					gender: genderValue,
					enter_time: ''
				};
				if (this.editingChildId) {
					const res = await updateChild(this.editingChildId, payload);
					if (res && res.status && res.status !== 100001) {
						uni.showToast({
							title: res.msg || '保存失败',
							icon: 'none'
						});
						return;
					}
				} else {
					const res = await createChild(payload);
					const newId = res && res.data && res.data.child_uid;
					if (newId) {
						this.editingChildId = newId;
						if (!this.selectedChildIds.includes(newId)) {
							this.selectedChildIds.push(newId);
						}
					}
				}
				await this.loadChildren();
				if (this.editingChildId) {
					const targetId = Number(this.editingChildId);
					if (!this.selectedChildIds.includes(targetId)) {
						this.selectedChildIds.push(targetId);
					}
				}
				this.showAddChild = false;
			} catch (e) {
				uni.showToast({
					title: '保存失败',
					icon: 'none'
				});
			}
		},
		async removeChild(child) {
			if (!child) return;
			try {
				const res = await deleteChild(child.child_uid);
				if (res && res.status && res.status !== 100002) {
					uni.showToast({
						title: res.msg || '删除失败',
						icon: 'none'
					});
					return;
				}
				await this.loadChildren();
				this.selectedChildIds = this.selectedChildIds.filter((id) => Number(id) !== Number(child.child_uid));
			} catch (e) {
				uni.showToast({
					title: '删除失败',
					icon: 'none'
				});
			}
		},
		toggleChildSelect(child) {
			if (!child) return;
			const id = Number(child.child_uid);
			const idx = this.selectedChildIds.findIndex((item) => Number(item) === id);
			if (idx >= 0) {
				this.selectedChildIds.splice(idx, 1);
			} else {
				this.selectedChildIds.push(id);
			}
		},
		isChildSelected(child) {
			if (!child) return false;
			const id = Number(child.child_uid);
			return this.selectedChildIds.some((item) => Number(item) === id);
		},
		formatChildMeta(child) {
			if (!child) return '请补充资料';
			const gender = this.normalizeGender(child.gender) === 'female' ? '女' : '男';
			const birth = child.birth_date || '';
			return birth ? `${gender} · ${birth}` : `${gender} · 生日待补`;
		},
		getClasstype() {
			const productInfo = this.productInfo || {};
			if (productInfo.classtype) return String(productInfo.classtype).toUpperCase();
			if (productInfo.class_type) return String(productInfo.class_type).toUpperCase();
			const name = `${productInfo.store_name || ''}${this.selectedSpec || ''}${this.selectedAttr || ''}`;
			const match = name.match(/[A-F]/i);
			if (match) return match[0].toUpperCase();
			const map = {
				17: 'A',
				15: 'B',
				13: 'C',
				11: 'D',
				10: 'E',
				9: 'F'
			};
			return map[Number(this.productId)] || 'A';
		},
		formatDate(value) {
			const date = value instanceof Date ? value : new Date(value);
			if (isNaN(date.getTime())) return '';
			const y = date.getFullYear();
			const m = String(date.getMonth() + 1).padStart(2, '0');
			const d = String(date.getDate()).padStart(2, '0');
			return `${y}-${m}-${d}`;
		},
		addDays(dateStr, days) {
			const date = new Date(dateStr);
			if (isNaN(date.getTime())) return '';
			date.setDate(date.getDate() + days);
			return this.formatDate(date);
		},
		async getHolidayDates() {
			if (!Object.keys(this.termDateMap || {}).length) {
				await this.loadTermMap();
			}
			const startTerm = this.startIndex + 1;
			const endTerm = this.isHolidayMonth ? this.endIndex + 1 : startTerm;
			const start = this.termDateMap[startTerm] || {};
			const end = this.termDateMap[endTerm] || {};
			return {
				startdate: start.startdate || '',
				deadline: end.deadline || ''
			};
		},
		async resolveCourseDates() {
			if (this.isHoliday) {
				return await this.getHolidayDates();
			}
			if (!this.courseStartDate) return { startdate: '', deadline: '' };
			const days = Math.max(1, this.selectedWeeks) * 7 - 1;
			return {
				startdate: this.courseStartDate,
				deadline: this.addDays(this.courseStartDate, days)
			};
		},
		getFromValue() {
			let from = '';
			// #ifdef H5
			from = this.$wechat && this.$wechat.isWeixin && this.$wechat.isWeixin() ? 'weixin' : 'weixinh5';
			// #endif
			// #ifdef MP
			from = 'routine';
			// #endif
			// #ifdef APP-PLUS
			from = 'app';
			// #endif
			return from;
		},
		async submitOrder() {
			if (!this.productId) return;
			if (!this.selectedChildIds.length) {
				uni.showToast({
					title: '请选择学员',
					icon: 'none'
				});
				return;
			}
			if (!this.isHoliday && !this.courseStartDate) {
				uni.showToast({
					title: '请选择开课时间',
					icon: 'none'
				});
				return;
			}
			const dates = await this.resolveCourseDates();
			if (!dates.startdate || !dates.deadline) {
				uni.showToast({
					title: '请先选择对应期次',
					icon: 'none'
				});
				return;
			}
			const courseParams = {
				classtype: this.getClasstype(),
				startdate: dates.startdate,
				deadline: dates.deadline,
				bigclass: 1
			};
			if (this.selectedChildIds.length === 1) {
				try {
					const cartRes = await postCartAdd({
						productId: this.productId,
						cartNum: 1,
						new: 1,
						uniqueId: this.attrUnique || '',
						virtual_type: this.productInfo.virtual_type,
						bigclass: 1
					});
					const cartId = cartRes && cartRes.data && cartRes.data.cartId;
					if (!cartId) return;
					const childId = this.selectedChildIds[0];
					const query = [
						`cartId=${cartId}`,
						'new=1',
						`classtype=${encodeURIComponent(courseParams.classtype || '')}`,
						`startdate=${encodeURIComponent(courseParams.startdate || '')}`,
						`deadline=${encodeURIComponent(courseParams.deadline || '')}`,
						`bigclass_child_uid=${encodeURIComponent(childId)}`,
						'bigclass=1'
					].join('&');
					uni.navigateTo({
						url: `/pages/goods/order_confirm/index?${query}`
					});
				} catch (e) {
					uni.showToast({
						title: '下单失败',
						icon: 'none'
					});
				}
				return;
			}
			try {
				const cartRes = await postCartAdd({
					productId: this.productId,
					cartNum: this.selectedChildIds.length,
					new: 1,
					uniqueId: this.attrUnique || '',
					virtual_type: this.productInfo.virtual_type,
					bigclass: 1
				});
				const cartId = cartRes && cartRes.data && cartRes.data.cartId;
				if (!cartId) return;
				const query = [
					`cartId=${cartId}`,
					'new=1',
					`classtype=${encodeURIComponent(courseParams.classtype || '')}`,
					`startdate=${encodeURIComponent(courseParams.startdate || '')}`,
					`deadline=${encodeURIComponent(courseParams.deadline || '')}`,
					`child_uids=${encodeURIComponent(this.selectedChildIds.join(','))}`,
					'bigclass=1'
				].join('&');
				uni.navigateTo({
					url: `/pages/goods/order_confirm/index?${query}`
				});
			} catch (e) {
				uni.showToast({
					title: (e && (e.message || e.msg || e.errMsg || e)) || '下单失败',
					icon: 'none'
				});
			}
		},
		updateByClientX(clientX) {
			if (!this.isHoliday) return;
			const relative = clientX - this.trackLeft;
			const maxIndex = this.totalSlots - 1;
			if (this.activeHandle === 'start') {
				let boundary = Math.round(relative / this.stepWidth);
				boundary = Math.max(0, Math.min(maxIndex, boundary));
				if (this.isHolidayWeek) {
					this.startIndex = boundary;
					this.endIndex = boundary;
				} else if (this.isHolidayMonth) {
					const start = Math.min(boundary, this.totalSlots - 4);
					this.startIndex = start;
					this.endIndex = Math.min(start + 3, maxIndex);
				} else {
					this.startIndex = Math.min(boundary, this.endIndex);
				}
			} else {
				let boundary = Math.round(relative / this.stepWidth);
				boundary = Math.max(1, Math.min(this.totalSlots, boundary));
				const newEnd = Math.max(boundary - 1, this.startIndex);
				if (this.isHolidayMonth) {
					const end = Math.max(3, Math.min(newEnd, maxIndex));
					this.endIndex = end;
					this.startIndex = Math.max(0, end - 3);
				} else {
					this.endIndex = newEnd;
				}
			}
		},
		applySnap() {
			const length = this.endIndex - this.startIndex + 1;
			const options = [1, 4, 8];
			if (this.totalSlots && !options.includes(this.totalSlots)) {
				options.push(this.totalSlots);
			}
			let nearest = options[0];
			options.forEach((opt) => {
				if (Math.abs(opt - length) < Math.abs(nearest - length)) {
					nearest = opt;
				}
			});
			if (this.isHolidayWeek) {
				this.endIndex = this.startIndex;
				return;
			}
			if (this.isHolidayMonth) {
				this.endIndex = Math.min(this.startIndex + 3, this.totalSlots - 1);
				return;
			}
			if (nearest === 8) {
				this.startIndex = 0;
				this.endIndex = 7;
				return;
			}
			if (nearest === 1) {
				this.endIndex = this.startIndex;
				return;
			}
			const overlapAug = Math.max(0, Math.min(this.endIndex, 3) - Math.max(this.startIndex, 0) + 1);
			const overlapSep = Math.max(0, Math.min(this.endIndex, 7) - Math.max(this.startIndex, 4) + 1);
			if (overlapAug >= overlapSep) {
				this.startIndex = 0;
				this.endIndex = 3;
			} else {
				this.startIndex = 4;
				this.endIndex = 7;
			}
		},
		loadProductPrices() {
			if (this.priceLoading) return;
			this.priceLoading = true;
			getProductDetail(this.priceProductId)
				.then((res) => {
					const productAttr = (res.data && res.data.productAttr) || [];
					const productValue = (res.data && res.data.productValue) || {};
					const values = Array.isArray(productValue) ? productValue : Object.values(productValue);
					const valueMap = Array.isArray(productValue) ? {} : productValue;
					const attrValues = [];
					productAttr.forEach((attr) => {
						(attr.attr_values || []).forEach((val) => {
							if (val) attrValues.push(val);
						});
					});
					const normalize = (val) => String(val || '').replace(/\s/g, '');
					const detectType = (val) => {
						const text = normalize(val);
						if (!text) return '';
						if (text.includes('周')) {
							if (/[1一]/.test(text)) return 'week';
							if (/[4四]/.test(text)) return 'month';
							if (/[8八]/.test(text)) return 'twoMonth';
						}
						if (text.includes('月')) {
							if (/[1一]/.test(text)) return 'month';
							if (/[2二两]/.test(text)) return 'twoMonth';
						}
						return '';
					};
					const matchPriceByValue = (targetValue) => {
						const target = normalize(targetValue);
						if (!target) return '';
						if (!Array.isArray(productValue)) {
							for (const key in valueMap) {
								const item = valueMap[key] || {};
								const suk = normalize(item.suk || key);
								if (suk.split(',').some((part) => normalize(part) === target)) {
									return item.price || item.vip_price || '';
								}
								if (suk.includes(target)) {
									return item.price || item.vip_price || '';
								}
							}
						}
						for (let i = 0; i < values.length; i += 1) {
							const item = values[i] || {};
							const suk = normalize(item.suk);
							if (!suk) continue;
							if (suk.split(',').some((part) => normalize(part) === target)) {
								return item.price || item.vip_price || '';
							}
							if (suk.includes(target)) {
								return item.price || item.vip_price || '';
							}
						}
						return '';
					};
					const priceMap = { week: '', month: '', twoMonth: '' };
					attrValues.forEach((val) => {
						const type = detectType(val);
						if (type && !priceMap[type]) {
							priceMap[type] = matchPriceByValue(val);
						}
					});
					if (!priceMap.week || !priceMap.month || !priceMap.twoMonth) {
						values.forEach((item) => {
							const type = detectType(item && item.suk);
							if (type && !priceMap[type]) {
								priceMap[type] = item.price || item.vip_price || '';
							}
						});
					}
					this.priceMap = priceMap;
				})
				.finally(() => {
					this.priceLoading = false;
				});
		},
		onBirthdayChange(e) {
			this.childBirthday = e.detail.value;
		},
		onGenderChange(e) {
			this.childGender = e.detail.value;
		},
		onCourseStartDateChange(e) {
			this.courseStartDate = e.detail.value;
		}
	}
};
</script>

<style lang="scss" scoped>
.course-booking {
	min-height: 100vh;
	background: #f7f8fa;
	padding: 120rpx 24rpx 160rpx;
	box-sizing: border-box;
}

.section + .section {
	margin-top: 24rpx;
}

.section-title {
	font-size: 30rpx;
	font-weight: 600;
	color: #1f2937;
	margin-bottom: 16rpx;
}

.duration-card {
	background: #ffffff;
	border-radius: 20rpx;
	padding: 24rpx;
	box-shadow: 0 10rpx 30rpx rgba(15, 23, 42, 0.06);
}

.child-select {
	display: flex;
	flex-direction: column;
	gap: 12rpx;
	margin-bottom: 12rpx;
}

.child-header {
	display: flex;
	align-items: center;
	justify-content: space-between;
}

.child-hint {
	font-size: 24rpx;
	color: #6b7280;
}

.child-list {
	display: grid;
	grid-template-columns: repeat(2, 1fr);
	gap: 16rpx;
}

.class-loading {
	font-size: 24rpx;
	color: #9ca3af;
	padding: 12rpx 0;
}

.class-list {
	display: grid;
	grid-template-columns: repeat(2, 1fr);
	gap: 16rpx;
}

.class-card {
	border-radius: 16rpx;
	border: 1rpx solid #e5e7eb;
	padding: 18rpx;
	background: #ffffff;
	display: flex;
	flex-direction: column;
	gap: 8rpx;
}

.class-card.active {
	border-color: #111827;
	box-shadow: 0 10rpx 24rpx rgba(15, 23, 42, 0.08);
}

.class-name {
	font-size: 28rpx;
	font-weight: 600;
	color: #111827;
}

.class-meta {
	font-size: 22rpx;
	color: #6b7280;
}

.class-empty {
	font-size: 24rpx;
	color: #9ca3af;
	padding: 12rpx 0;
}

.child-card {
	border-radius: 16rpx;
	border: 1rpx solid #e5e7eb;
	padding: 18rpx;
	display: flex;
	flex-direction: column;
	gap: 12rpx;
	background: #f9fafb;
}

.child-card.active {
	border-color: #111827;
	background: #ffffff;
	box-shadow: 0 10rpx 24rpx rgba(15, 23, 42, 0.08);
}

.child-info {
	display: flex;
	flex-direction: column;
	gap: 6rpx;
}

.child-name {
	font-size: 28rpx;
	color: #111827;
	font-weight: 600;
}

.child-meta {
	font-size: 22rpx;
	color: #6b7280;
}

.child-actions-inline {
	display: flex;
	align-items: center;
	gap: 10rpx;
	flex-wrap: wrap;
}

.child-tag {
	padding: 4rpx 12rpx;
	border-radius: 999rpx;
	background: #111827;
	color: #ffffff;
	font-size: 20rpx;
}

.child-mini-btn {
	padding: 4rpx 12rpx;
	border-radius: 999rpx;
	background: #eef2ff;
	color: #4f46e5;
	font-size: 20rpx;
}

.child-mini-btn.danger {
	background: #fee2e2;
	color: #b91c1c;
}

.add-card {
	border: 1rpx dashed #cbd5f5;
	background: #ffffff;
	align-items: center;
	justify-content: center;
	min-height: 150rpx;
}

.add-plus {
	font-size: 44rpx;
	color: #4f46e5;
	line-height: 1;
}

.add-text {
	font-size: 24rpx;
	color: #4f46e5;
}

.child-guide {
	font-size: 22rpx;
	color: #6b7280;
}

.child-save-bar {
	display: flex;
	gap: 12rpx;
	margin-top: 12rpx;
}

.child-save-btn,
.child-cancel-btn {
	flex: 1;
	text-align: center;
	padding: 16rpx 0;
	border-radius: 999rpx;
	font-size: 26rpx;
}

.child-save-btn {
	background: #111827;
	color: #ffffff;
}

.child-cancel-btn {
	background: #e5e7eb;
	color: #374151;
}

.spec-toggle {
	display: flex;
	align-items: center;
	justify-content: space-between;
	margin-bottom: 20rpx;
}

.spec-label {
	font-size: 26rpx;
	color: #6b7280;
}

.spec-options {
	display: flex;
	gap: 12rpx;
}

.spec-option {
	padding: 10rpx 22rpx;
	border-radius: 999rpx;
	border: 1rpx solid #e5e7eb;
	color: #4b5563;
	font-size: 24rpx;
}

.spec-option.active {
	background: #111827;
	color: #ffffff;
	border-color: #111827;
}

.top-bar {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	height: 88rpx;
	display: flex;
	align-items: center;
	padding: 0 24rpx;
	background: #f7f8fa;
	z-index: 20;
}

.back-btn {
	display: flex;
	align-items: center;
	height: 64rpx;
	padding: 0 18rpx;
	background: #ffffff;
	border-radius: 32rpx;
	box-shadow: 0 8rpx 20rpx rgba(15, 23, 42, 0.08);
	color: #111827;
}

.back-btn .iconfont {
	font-size: 28rpx;
}

.back-text {
	margin-left: 8rpx;
	font-size: 24rpx;
}

.duration-track {
	position: relative;
	height: 14rpx;
	background: #e5e7eb;
	border-radius: 999rpx;
	user-select: none;
}

.duration-nodes {
	position: absolute;
	top: 50%;
	left: 0;
	right: 0;
	height: 0;
	z-index: 0;
}

.duration-node {
	position: absolute;
	top: 50%;
	width: 10rpx;
	height: 10rpx;
	border-radius: 50%;
	background: #d1d5db;
	transform: translate(-50%, -50%);
}

.duration-range {
	position: absolute;
	top: 0;
	height: 14rpx;
	background: #22c55e;
	border-radius: 999rpx;
	z-index: 1;
}

.duration-handle {
	position: absolute;
	top: 50%;
	width: 36rpx;
	height: 36rpx;
	border-radius: 50%;
	background: #ffffff;
	border: 4rpx solid #22c55e;
	transform: translate(-50%, -50%);
	box-shadow: 0 6rpx 16rpx rgba(34, 197, 94, 0.25);
	z-index: 2;
}

.duration-months {
	display: flex;
	justify-content: space-between;
	margin-top: 18rpx;
	font-size: 22rpx;
	color: #6b7280;
}

.month-label {
	flex: 1;
	text-align: center;
}

.selection-info {
	margin-top: 10rpx;
	font-size: 24rpx;
	color: #374151;
	text-align: center;
}

.form-card {
	background: #ffffff;
	border-radius: 20rpx;
	padding: 10rpx 24rpx;
	box-shadow: 0 10rpx 30rpx rgba(15, 23, 42, 0.06);
}

.form-item {
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 22rpx 0;
	border-bottom: 1rpx solid #f1f5f9;
}

.buy-bar {
	position: fixed;
	left: 0;
	right: 0;
	bottom: 0;
	padding: 16rpx 24rpx 24rpx;
	background: #f7f8fa;
	z-index: 20;
}

.buy-button {
	height: 96rpx;
	border-radius: 48rpx;
	background: #22c55e;
	display: flex;
	align-items: center;
	justify-content: space-between;
	padding: 0 32rpx;
	color: #ffffff;
	font-size: 30rpx;
	font-weight: 600;
	box-shadow: 0 14rpx 30rpx rgba(34, 197, 94, 0.3);
}

.buy-meta {
	display: flex;
	align-items: center;
	gap: 12rpx;
}

.buy-text {
	margin-right: 0;
}

.buy-count {
	font-size: 26rpx;
	font-weight: 500;
	opacity: 0.9;
}

.buy-price {
	font-size: 32rpx;
}

.form-item:last-child {
	border-bottom: none;
}

.label {
	font-size: 26rpx;
	color: #1f2937;
}

input {
	flex: 1;
	text-align: right;
	font-size: 26rpx;
	color: #111827;
}

.placeholder {
	color: #cbd5f5;
}

.picker-value {
	font-size: 26rpx;
	color: #111827;
}

.gender-group {
	display: flex;
	align-items: center;
	gap: 24rpx;
}

.gender-item {
	display: flex;
	align-items: center;
	font-size: 26rpx;
	color: #111827;
}

radio {
	transform: scale(0.85);
	margin-right: 8rpx;
}
</style>
