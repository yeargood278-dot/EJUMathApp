<template>
	<view class="container">
		<view class="header-area">
			<text class="app-title">EJU 文科数学</text>
			<text class="app-subtitle">EJU Liberal Arts Mathematics Syllabus</text>
		</view>

		<view class="grid-container">
			<view 
				class="chapter-card" 
				v-for="(item, index) in chapterList" 
				:key="index"
				:style="{ background: item.color }"
				@click="goToChapter(item.id)"
			>
				<text class="jp-title">{{ item.jp }}</text>
				<text class="cn-title">{{ item.cn }}</text>
			</view>
		</view>

		<view class="footer">
			<text>版权所有：春风创意工作室</text>
			<text>著作権所有：春風クリエイティブスタジオ</text>
		</view>
	</view>
</template>

<script>
	// 引入数据
	import { chapters } from '@/common/courseData.js';

	export default {
		data() {
			return {
				chapterList: chapters
			}
		},
		methods: {
			goToChapter(id) {
				// 跳转到详情页，并传递章节ID
				uni.navigateTo({
					url: `/pages/chapter/chapter?id=${id}`
				});
			}
		}
	}
</script>

<style>
	/* 将原来的 CSS 移植过来，注意单位 px 可以改为 rpx 以适应手机屏幕 */
	page { background-color: #f4f8fb; }
	.container { padding: 20px; padding-bottom: 40px; display: flex; flex-direction: column; align-items: center; }
	
	.header-area { margin: 40rpx 0; text-align: center; }
	.app-title { font-size: 48rpx; color: #2c3e50; font-weight: bold; display: block; margin-bottom: 10rpx; letter-spacing: 2px;}
	.app-subtitle { font-size: 28rpx; color: #7f8c8d; }

	.grid-container {
		width: 100%;
		display: flex;
		flex-direction: column;
		gap: 25rpx;
	}

	.chapter-card {
		border-radius: 15px;
		padding: 40rpx 30rpx;
		color: white;
		box-shadow: 0 5px 15px rgba(0,0,0,0.1);
		position: relative;
		overflow: hidden;
	}
	
	/* 简单的点击效果 */
	.chapter-card:active { transform: scale(0.98); }

	.jp-title { font-size: 36rpx; font-weight: bold; display: block; margin-bottom: 10rpx; text-shadow: 1px 1px 2px rgba(0,0,0,0.1); }
	.cn-title { font-size: 26rpx; opacity: 0.9; display: block; }

	.footer { margin-top: 60rpx; text-align: center; color: #95a5a6; font-size: 24rpx; display: flex; flex-direction: column; gap: 5rpx;}
</style>
