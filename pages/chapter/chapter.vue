<template>
	<view class="content" :prop="currentData" :change:prop="module.renderMath">
		
		<view class="top-nav-area">
			<scroll-view scroll-x="true" class="nav-scroll" show-scrollbar="false">
				<view class="nav-container">
					<view class="nav-item home-btn" @click="goHome">🏠 主页</view>
					<view 
						class="nav-item" 
						v-for="i in 14" 
						:key="i"
						:class="{ active: cid === i }"
						@click="switchChapter(i)"
					>
						第{{i}}章
					</view>
				</view>
			</scroll-view>
		</view>

		<view class="container" v-if="currentData">
			<view class="chapter-header" :style="{ background: themeColor }">
				<text class="h1">{{ currentData.titleJP }}</text>
				<text class="p">{{ currentData.titleCN }}</text>
			</view>

			<view class="card">
				<view class="card-title">学習のポイント (考点)</view>
				<view v-for="(item, idx) in currentData.exam_points" :key="idx" class="item-row">
					<view class="jp-box">
						<view class="math-text" v-html="rawHtml(item.jp)"></view>
						<view class="btn-group">
							<view class="btn-mini btn-read" @click="module.speak(item.jp)">🔊</view>
							<view class="btn-mini btn-trans" @click="toggleTrans(idx, 'point')">译</view>
						</view>
					</view>
					<view class="cn-text" v-if="showStates.point[idx]">
						<view class="math-text" v-html="rawHtml(item.cn)"></view>
					</view>
				</view>
			</view>

			<view class="card">
				<view class="card-title">重要概念 (Concepts)</view>
				<view v-for="(item, idx) in currentData.concepts" :key="idx" class="item-row">
					<view class="jp-box">
						<text style="color:#e84393; font-weight:bold; margin-right:10px;">{{ item.term_jp }}</text>
						<view class="btn-mini btn-read" @click="module.speak(item.desc_jp)">🔊</view>
					</view>
					
					<view v-if="item.svg" class="svg-container" v-html="rawHtml(item.svg)"></view>
					
					<view class="math-text" style="margin-top:5px;" v-html="rawHtml(item.desc_jp)"></view>
					
					<view class="btn-trans-text" @click="toggleTrans(idx, 'concept')">查看中文释义</view>
					<view class="cn-text" v-if="showStates.concept[idx]">
						<view class="math-text" v-html="rawHtml(item.desc_cn)"></view>
					</view>
				</view>
			</view>

			<view class="card">
				<view class="card-title">公式と定理 (Formulas)</view>
				<view v-for="(item, idx) in currentData.formulas" :key="idx" class="item-row">
					<view style="font-weight:bold; color:#6c5ce7; margin-bottom:5px;">{{ item.name_jp }}</view>
					
					<view v-if="item.svg" class="svg-container" v-html="rawHtml(item.svg)"></view>

					<view class="math-block" v-html="rawHtml(item.content_jp)"></view>
					
					<view class="jp-box" style="margin-top:5px;">
						<text style="font-size:24rpx; color:#666;">Note:</text>
						<view class="math-text note-text" v-html="rawHtml(item.note_jp)"></view>
						<view class="btn-mini btn-read" @click="module.speak(item.note_jp)">🔊</view>
						<view class="btn-mini btn-trans" @click="toggleTrans(idx, 'formula')">译</view>
					</view>
					<view class="cn-text" v-if="showStates.formula[idx]">
						<view class="math-text" v-html="rawHtml(item.note_cn)"></view>
					</view>
				</view>
			</view>

			<view class="card">
				<view class="card-title">
					<text>例題解説 (Example)</text>
					<view class="btn-generate" @click="module.randomExample">⚡ 生成新题</view>
				</view>
				<view id="math-area-example" class="dynamic-zone"></view>
			</view>
			
			<view class="card">
				<view class="card-title">
					<text>確認テスト (Test)</text>
					<view class="btn-generate" style="background:#00b894;" @click="module.randomTest">⚡ 开始挑战</view>
				</view>
				<view id="math-area-test" class="dynamic-zone" style="background:#e3f2fd;">
					<text style="color:#666; text-align: center; display: block; padding: 20px;">点击按钮生成测试题</text>
				</view>
			</view>

			<view style="height: 50px;"></view>
		</view>
	</view>
</template>

<script>
	import { chapterDetails, chapters } from '@/common/courseData.js';

	export default {
		data() {
			return {
				cid: 1,
				currentData: null,
				themeColor: 'linear-gradient(135deg, #FF9A9E 0%, #FECFEF 100%)',
				showStates: { point: {}, concept: {}, formula: {} }
			}
		},
		onLoad(options) {
			this.loadChapter(options.id ? parseInt(options.id) : 1);
		},
		methods: {
			goHome() {
				uni.reLaunch({ url: '/pages/index/index' });
			},
			switchChapter(id) {
				// 使用 redirectTo 切换章节，避免页面栈堆积
				uni.redirectTo({
					url: `/pages/chapter/chapter?id=${id}`
				});
			},
			loadChapter(id) {
				this.cid = id;
				const data = chapterDetails[id];
				// 查找对应章节的颜色
				const meta = chapters.find(c => c.id === id);
				
				if(data) {
					this.currentData = data;
					if (meta) this.themeColor = meta.color;
					// 重置翻译状态
					this.showStates = { point: {}, concept: {}, formula: {} };
				}
			},
			toggleTrans(index, type) {
				this.$set(this.showStates[type], index, !this.showStates[type][index]);
			},
			rawHtml(str) {
				return str || ''; 
			}
		}
	}
</script>

<script module="module" lang="renderjs">
	// pages/chapter/chapter.vue 内部的 renderjs 模块
	import { chapterDetails } from '@/common/courseData.js';
	
	export default {
	    data() {
	        return {
	            isMathJaxReady: false,
	            cachedData: null
	        }
	    },
	    mounted() {
	        // 1. 配置 MathJax 选项
	        window.MathJax = {
	            tex: {
	                inlineMath: [['$', '$'], ['\\(', '\\)']],
	                displayMath: [['$$', '$$'], ['\\[', '\\]']],
	                processEscapes: true
	            },
	            options: {
	                enableMenu: false // 禁用右键菜单以适应移动端
	            },
	            startup: {
	                typeset: false // 禁用自动初始化
	            }
	        };
	
	        // 2. 动态加载 MathJax 脚本
	        if (typeof window.MathJax === 'undefined' || !window.MathJax.typesetPromise) {
	            const script = document.createElement('script');
	            script.src = "https://cdn.jsdelivr.net/npm/mathjax@3/es5/tex-mml-chtml.js";
	            script.async = true;
	            script.onload = () => {
	                this.isMathJaxReady = true;
	                if(this.cachedData) this.refreshAll();
	            };
	            document.head.appendChild(script);
	        } else {
	            this.isMathJaxReady = true;
	        }
	    },
	    methods: {
	        // 监听逻辑层数据变化
	        renderMath(newValue) {
	            if (newValue) {
	                this.cachedData = newValue;
	                this.refreshAll();
	            }
	        },
	
	        refreshAll() {
	            if (!this.isMathJaxReady) return;
	            
	            // 默认生成第一个例题
	            this.randomExample();
	            
	            // 核心：使用 nextTick 或 setTimeout 确保 DOM 已更新后再渲染公式
	            setTimeout(() => {
	                if (window.MathJax && window.MathJax.typesetPromise) {
	                    window.MathJax.typesetPromise().then(() => {
	                        console.log('MathJax渲染完成');
	                    }).catch((err) => console.error('MathJax渲染错误:', err));
	                }
	            }, 500); // 增加延迟以确保 Vue 完成 DOM 插入
	        },
	
	        runTypeset() {
	            // 局部动态生成内容后的手动渲染
	            this.$nextTick(() => {
	                if (window.MathJax && window.MathJax.typesetPromise) {
	                    window.MathJax.typesetPromise();
	                }
	            });
	        },
	
	        randomExample() {
	            let data = this.cachedData || chapterDetails[1];
	            if(data && data.pool_examples) {
	                const item = data.pool_examples[Math.floor(Math.random() * data.pool_examples.length)];
	                const el = document.getElementById('math-area-example');
	                if(el) {
	                    el.innerHTML = `
	                        <div class="math-q">${item.q_jp}</div>
	                        <div class="math-cn">${item.q_cn}</div>
	                        <div class="math-sol">解： ${item.sol}</div>
	                    `;
	                    this.runTypeset(); // 重新渲染新生成的公式
	                }
	            }
			},

			// 生成随机测试
			randomTest(event, ownerInstance) {
				let data = this.cachedData;
				if (!data) data = chapterDetails[1];

				if(data && data.pool_tests && data.pool_tests.length > 0) {
					const list = data.pool_tests;
					const item = list[Math.floor(Math.random() * list.length)];
					const el = document.getElementById('math-area-test');
					const uid = 'ans-' + Math.floor(Math.random() * 10000);
					
					if(el) {
						el.innerHTML = `
							<div style="margin-bottom:15px; font-weight:bold; font-size:16px;">Q. ${item.q}</div>
							<div id="btn-${uid}" style="text-align:center;">
								<button onclick="document.getElementById('${uid}').style.display='block'; this.style.display='none'" style="background:#00b894; color:white; border:none; padding:8px 20px; border-radius:20px; font-size:14px;">查看答案</button>
							</div>
							<div id="${uid}" style="display:none; margin-top:10px; text-align:center; padding:10px; background:#fff; border-radius:8px;">
								<div style="color:#0984e3; font-weight:bold; font-size:16px;">${item.a}</div>
							</div>
						`;
						this.runTypeset();
					}
				}
			},

			speak(text) {
				if(!text) return;
				let clean = text.replace(/\$.*?\$/g, "数式");
				window.speechSynthesis.cancel();
				let u = new SpeechSynthesisUtterance(clean);
				u.lang = 'ja-JP'; 
				u.rate = 0.9;
				window.speechSynthesis.speak(u);
			}
		}
	}
</script>

<style>
	page { background-color: #f4f8fb; }
	.content { padding-bottom: 50px; }
	
	/* 导航栏样式 */
	.top-nav-area {
		position: sticky;
		top: 0;
		z-index: 100;
		background: #fff;
		box-shadow: 0 2px 10px rgba(0,0,0,0.05);
	}
	.nav-scroll {
		width: 100%;
		white-space: nowrap;
		height: 50px;
	}
	.nav-container {
		display: flex;
		align-items: center;
		height: 50px;
		padding: 0 10px;
	}
	.nav-item {
		display: inline-flex;
		align-items: center;
		justify-content: center;
		height: 32px;
		padding: 0 15px;
		margin-right: 10px;
		background: #f0f2f5;
		border-radius: 16px;
		font-size: 14px;
		color: #555;
		font-weight: bold;
	}
	.nav-item.active {
		background: #2d3436;
		color: #fff;
	}
	.home-btn {
		background: #00b894;
		color: white;
	}

	.nav-bar { 
		padding: 10px 20px; 
		display: flex; 
		align-items: center; 
		font-size: 14px; 
		color: #666;
	}
	.back-btn { padding: 5px; }
	
	.container { padding: 30rpx; }
	.chapter-header { padding: 60rpx 30rpx; border-radius: 20rpx; color: white; margin-bottom: 40rpx; box-shadow: 0 5px 15px rgba(0,0,0,0.1); text-align: center; }
	.h1 { font-size: 40rpx; font-weight: bold; display: block; margin-bottom: 15rpx; text-shadow: 1px 1px 3px rgba(0,0,0,0.2); }
	.p { font-size: 28rpx; opacity: 0.95; }

	.card { background: white; border-radius: 15px; padding: 30rpx; margin-bottom: 30rpx; box-shadow: 0 4px 12px rgba(0,0,0,0.05); }
	.card-title { font-size: 34rpx; font-weight: bold; border-left: 8rpx solid #ff758c; padding-left: 20rpx; margin-bottom: 30rpx; display: flex; justify-content: space-between; align-items: center; color: #2c3e50; }
	
	.item-row { border-bottom: 1px dashed #eee; padding-bottom: 25rpx; margin-bottom: 25rpx; }
	.item-row:last-child { border-bottom: none; margin-bottom: 0; }
	
	.jp-box { display: flex; flex-wrap: wrap; align-items: center; margin-bottom: 10rpx; }
	.math-text { font-size: 30rpx; color: #333; line-height: 1.6; word-wrap: break-word; }
	.note-text { font-size: 26rpx; color: #666; margin-left: 5px; display: inline-block; }

	.btn-group { display: inline-flex; align-items: center; }
	.btn-mini { display: inline-flex; align-items: center; justify-content: center; width: 24px; height: 24px; border-radius: 50%; font-size: 12px; color: white; margin-left: 8rpx; }
	.btn-read { background: #fab1a0; }
	.btn-trans { background: #74b9ff; }
	.btn-trans-text { font-size: 24rpx; color: #999; margin-top: 10rpx; text-decoration: underline; }
	
	.cn-text { background: #f8f9fa; padding: 15rpx; border-radius: 8rpx; font-size: 28rpx; color: #555; margin-top: 15rpx; }
	
	.btn-generate { font-size: 24rpx; background: #9b59b6; color: white; padding: 10rpx 25rpx; border-radius: 30rpx; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
	.btn-generate:active { transform: scale(0.95); opacity: 0.9; }

	.math-block { margin: 20rpx 0; padding: 20rpx; background: #fdfdfd; border-radius: 12rpx; overflow-x: auto; -webkit-overflow-scrolling: touch; }
    .dynamic-zone:empty { display: none; }
	.svg-container { width: 100%; display: flex; justify-content: center; margin: 20rpx 0; padding: 20rpx; background-color: #f9f9f9; border-radius: 12rpx; }
	.svg-container :deep(svg) { width: 60% !important; height: auto !important; max-width: 400rpx; filter: drop-shadow(0 2px 4px rgba(0,0,0,0.1)); }
	@media (min-width: 768px) { .svg-container :deep(svg) { max-width: 500rpx; } }
	</style>