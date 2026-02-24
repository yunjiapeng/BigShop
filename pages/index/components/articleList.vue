<template>
	<!-- 新闻列表 -->
	<view v-show="!isSortType" :style="[articleWrapperStyle]">
		<view class="articleList" :class="{
			large: dataConfig.styleConfig.tabVal == 0,
			small: dataConfig.styleConfig.tabVal == 2,
		}" v-if="filteredArticleList.length">
			<component :is="disableNavigate ? 'view' : 'navigator'" v-for="(item,index) in filteredArticleList" :key="index" v-bind="navProps(item)" :style="[articleItemStyle]" class="item" @click="handleItemClick(item)">
				<view v-if="dataConfig.styleConfig.tabVal != 0" class="image-wrap" :style="[imageWrapStyle]">
					<easy-loadimage :imageSrc="item.image_input[0]" :borderRadius="borderRadius" width="100%" height="100%"></easy-loadimage>
				</view>
				<view class="text-wrap">
					<view class="title-wrap">
						<view class="title" :class="{
							line1: dataConfig.styleConfig.tabVal != 1,
							line2: dataConfig.styleConfig.tabVal == 1,
						}" :style="[titleStyle]">{{ item.title }}</view>
					</view>
					<view v-if="dataConfig.styleConfig.tabVal == 0" class="image-wrap" :style="[imageWrapStyle]">
						<easy-loadimage :imageSrc="item.image_input[0]" :borderRadius="borderRadius" width="100%" height="100%"></easy-loadimage>
					</view>
					<view class="time-wrap"></view>
				</view>
			</component>
		</view>
	</view>
</template>

<script>
	import {
		getArticleList
	} from '@/api/api.js';
	import dayjs from '@/plugin/dayjs/dayjs.min.js';
	export default {
		name: 'articleList',
		props: {
			dataConfig: {
				type: Object,
				default: () => {}
			},
			isSortType: {
				type: String | Number,
				default: 0
			},
			disableNavigate: {
				type: Boolean,
				default: false
			},
			excludeIds: {
				type: Array,
				default: () => []
			}
		},
		data() {
			return {
				articleList: [],
			}
		},
		computed: {
			isLike(){
				let like = true
				like = this.dataConfig.checkboxList.type.includes(2);
				return like
			},
			unifiedRadius() {
				const extra = 4;
				let borderRadius = `${(this.dataConfig.fillet.val + extra) * 2}rpx`;
				if (this.dataConfig.fillet.type) {
					borderRadius =
						`${(this.dataConfig.fillet.valList[0].val + extra) * 2}rpx ${(this.dataConfig.fillet.valList[1].val + extra) * 2}rpx ${(this.dataConfig.fillet.valList[3].val + extra) * 2}rpx ${(this.dataConfig.fillet.valList[2].val + extra) * 2}rpx`;
				}
				return borderRadius;
			},
			filteredArticleList() {
				const excludeSet = new Set((this.excludeIds || []).map((id) => String(id)));
				return (this.articleList || []).filter((item) => !excludeSet.has(String(item.id)));
			},
			borderRadius() {
				return this.unifiedRadius;
			},
			timeStyle() {
				return {
					'color': this.dataConfig.timeColor.color[0].item,
				};
			},
			iconEyesStyle() {
				return {
					'color': this.dataConfig.browseColor.color[0].item,
				};
			},
			// iconLikeStyle() {
			// 	return {
			// 		'color': this.dataConfig.likeColor.color[0].item,
			// 	};
			// },
			numberStyle() {
				return {
					'color': this.dataConfig.statisticColor.color[0].item,
				};
			},
			articleItemStyle() {
				let borderRadius = this.unifiedRadius;
				return {
					'border-radius': borderRadius,
					'background': '#FFFFFF',
					'box-sizing': 'border-box',
					'border': '2rpx solid var(--panel-border, #B7DBB6)',
				};
			},
			imageWrapStyle() {
				return {
					'border-radius': this.unifiedRadius,
					'overflow': 'hidden',
				};
			},
			articleWrapperStyle() {
				return {
					'padding': `${this.dataConfig.topConfig.val * 2}rpx ${this.dataConfig.prConfig.val * 2}rpx ${this.dataConfig.bottomConfig.val * 2}rpx`,
					'margin-top': `${this.dataConfig.mbConfig.val * 2}rpx`,
					'background': this.dataConfig.bottomBgColor.color[0].item,
				};
			},
			titleStyle() {
				let styleObject = {
					'color': '#000000',
				};
				if (!this.dataConfig.nameConfig.tabVal) {
					styleObject['font-weight'] = 'bold';
				}
				return styleObject;
			}
		},
		mounted() {
			this.getCidArticle();
		},
		methods: {
			navProps(item) {
				if (this.disableNavigate) return {};
				return {
					url: '/pages/extension/news_details/index?id=' + item.id,
					'hover-class': 'none'
				};
			},
			handleItemClick(item) {
				if (this.disableNavigate) {
					this.$emit('open', item);
				}
			},
			getCidArticle: function() {
				let limit = this.$config.LIMIT;
				getArticleList(this.dataConfig.selectConfig.activeValue || 0, {
					page: 1,
					// limit: this.dataConfig.numConfig.val >= limit ? limit : this.dataConfig.numConfig.val
					limit: Math.max(limit, 9999)
				}).then(res => {
					if (this.dataConfig.styleConfig.tabVal == 2) {
						res.data.forEach(item => {
							item.add_time = dayjs(item.add_time).format('MM-DD HH:mm')
						})
					}
					this.articleList = res.data || [];
				});
			},
		}
	}
</script>

<style lang="scss">
	.articleList {
		.item {
			display: flex;
			padding: 20rpx;
			margin: 0;
			background: #FFFFFF;
		}

		.text-wrap {
			flex: 1;
			display: flex;
			flex-direction: column;
			padding: 0 0 6rpx 24rpx;
		}

		.title-wrap {
			flex: 1;
		}

		.title {
			font-size: 30rpx;
			line-height: 42rpx;
			color: #333333;
		}

		.label-wrap {
			margin-top: 16rpx;
			font-size: 0;
		}

		.label {
			display: inline-flex;
			flex-wrap: nowrap;
			height: 28rpx;
			padding: 0 6rpx;
			border: 0;
			border-radius: 6rpx;
			font-size: 20rpx;
			color: #666666;
		}

		.image-wrap {
			width: 240rpx;
			height: 152rpx;
		}

		.time-wrap {
			display: flex;
			font-size: 24rpx;
			line-height: 34rpx;
			color: #999999;

			.time-wrap-item:first-child {
				flex: 1;
			}
		}

		.like-wrap {
			display: flex;
		}

		.view {
			flex: 1;
			&.on{
				margin-right: 0;
			}
		}

		.iconfont {
			margin-right: 8rpx;
			font-size: 28rpx;
		}

		&.large {
			.item {
				flex-direction: column;
				padding: 0;
			}

			.text-wrap {
				padding: 32rpx 24rpx;
			}

			.image-wrap {
				width: 100%;
				height: 320rpx;
				margin-top: 24rpx;
			}

			.time-wrap {
				margin-top: 24rpx;
			}
		}

		&.small {
			display: flex;
			flex-wrap: wrap;
			justify-content: space-between;

			.item {
				flex-direction: column;
				width: 342rpx;
				padding: 0;
				margin-bottom: 20rpx;
			}

			.text-wrap {
				padding: 20rpx 20rpx 18rpx;
			}

			.image-wrap {
				width: 342rpx;
				height: 216rpx;
			}

			.label-wrap {
				margin-top: 16rpx;
			}

			.time-wrap {
				margin-top: 16rpx;
			}

			.like-wrap {
				justify-content: space-between;
			}
		}
	}
</style>
