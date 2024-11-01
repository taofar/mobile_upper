<template>
	<!-- <view class="content"> -->
		<!-- #ifdef APP-PLUS || H5 -->
		<view :prop="localOption" :change:prop="echarts.updateEcharts" :id="id" class="echarts"></view>
		<!-- #endif -->
		<!-- #ifndef APP-PLUS || H5 -->
		<view>非 APP、H5 环境不支持</view>
		<!-- #endif -->
	<!-- </view> -->
</template>

<script>
	export default {
		data() {
			return {
				// chartId: 'echart',
				localOption: {
					title: {
						// text: '示波器'
					},
					tooltip: {},
					legend: {
						data: ['销量']
					},
					xAxis: {
						data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
					},
					yAxis: {},
					series: [{
						name: '销量',
						type: 'line',
						data: [5, 20, 36, 10, 10, 20]
					}]
				}
			}
		},
		props:{
			id: String,
			option: {
				type: Object,
				default:()=>{}
			}
		},
		watch: {
			'option': function(param) {
				this.localOption = param;
			}
		},
		onLoad() {

		},
		created() {
			this.localOption = this.option;
			this.chartId = this.id;
		},
		methods: {
			changeOption() {
				const data = this.option.series[0].data
				// 随机更新示例数据
				data.forEach((item, index) => {
					data.splice(index, 1, Math.random() * 40)
				})
			},
			onViewClick(options) {
				console.log(options)
			}
		}
	}
</script>

<script module="echarts" lang="renderjs">
	let myChart
	export default {
		mounted() {
			if (typeof window.echarts === 'function') {
				this.initEcharts()
			} else {
				// 动态引入较大类库避免影响页面展示
				const script = document.createElement('script')
				// view 层的页面运行在 www 根目录，其相对路径相对于 www 计算
				script.src = 'static/echarts.js'
				script.onload = this.initEcharts.bind(this)
				document.head.appendChild(script)
			}
		},
		methods: {
			initEcharts() {
				// console.log(this.id);
				myChart = echarts.init(document.getElementById(this.id))
				// 观测更新的数据在 view 层可以直接访问到
				myChart.setOption(this.localOption)
			},
			updateEcharts(newValue, oldValue, ownerInstance, instance) {
				// 监听 service 层数据变更
				myChart.setOption(newValue)
			},
			onClick(event, ownerInstance) {
				// 调用 service 层的方法
				console.log(event);
				ownerInstance.callMethod('onViewClick', {
					test: event
				})
			}
		}
	}
</script>

<style>
	.echarts {
		top: 0px;
		left: 0px;
		width: 100%;
		position: absolute;
		height: 100%;
		padding: 0 0;
		margin: 0 0;
		/* background-color: #007AFF; */
		/* border: #000000 solid 2rpx; */
	}
</style>
