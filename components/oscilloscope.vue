<template>
	<view class="content">
		<view class="oscilloscope-container">
			<echarts id="oscilloscope" :option="option"></echarts>
		</view>
	</view>
</template>

<script>
	import echarts from './echarts.vue'

	function randomData() {
		now = new Date(+now + oneDay);
		value = value + Math.random() * 21 - 10;
		return {
			name: now.toString(),
			value: [
				[now.getFullYear(), now.getMonth() + 1, now.getDate()].join('/'),
				Math.round(value)
			]
		};
	}
	let data = [];
	let now = new Date(1997, 9, 3);
	let oneDay = 24 * 3600 * 1000;
	let value = Math.random() * 1000;
	for (var i = 0; i < 1; i++) {
		data.push(randomData());
	}
	export default {
		data() {
			return {
				startTime: 0,
				originTime: 0,
				timer: null,
				sampleTime: 100, // 100ms
				timeLineLength: 100 * 1000, // 100 * 1000ms
				channels: {},
				option: {
					title: {
						text: 'oscilloscope'
					},
					toolbox: {
						feature: {
							dataZoom: {
								yAxisIndex: 'none'
							},
							restore: {},
							saveAsImage: {}
						},
						itemSize: 20
					},
					legend: {
						type: 'scroll',
						width: '60%',
						itemHeight: 20,
						itemWidth: 30
					},
					// animationDuration:100,
					tooltip: {
						trigger: 'axis',
						formatter: function(params) {
							params = params[0];
							var date = new Date(params.name);
							return (
								date.getDate() +
								'/' +
								(date.getMonth() + 1) +
								'/' +
								date.getFullYear() +
								' : ' +
								params.value[1]
							);
						},
						axisPointer: {
							animation: false
						}
					},
					xAxis: {
						type: 'time',
						axisLabel: {
							show: false
						},
						axisPointer: {
							show:true
						},
					},
					yAxis: {
						type: 'value',
						boundaryGap: [0, '100%'],
						axisPointer: {
							show:true
						},
						splitLine: {
							show: false
						}
					},
					dataZoom: [{
						show: true,
						realtime: true,
						// start: 90,
						// end: 100,
						// maxValueSpan: 10000
					}],
					series: []
				}
			}
		},
		// channel:Array: time value
		props: {
			signalList: {
				type: Array,
				default: () => [{}]
			},
			samplingEvent: String
		},
		beforeMount() {
			console.log(this.samplingEvent);
			if (this.samplingEvent) uni.$on(this.samplingEvent, (data) => {
				// console.log(data);
				// 事件携带的参数类型为 {signalName, timestamp, value}
				if (this.originTime == 0) this.originTime = data.time - 2 * this.sampleTime;
				if (this.startTime == 0) this.startTime = this.originTime;
				// 找到对应的通道
				// 数据模型
				// channels:{ "name": {
				// 	lastTime: number,	// 上次采样时间
				//  realTime: number,
				//  type: scatter || pop
				// 	length: number, 	// 数据长度
				// 	list: [[time, value], [time1, value1]]	// 数据列表
				// },}
				if (!this.channels.hasOwnProperty(data.signalName)) {
					this.channels[data.signalName] = {
						lastTime: 0,
						type: (data.type === 'scatter') ? 'scatter' : 'line',
						list: [],
						length: 0
					};
				}
				// 判断是否在最小采样时间内重复采样
				// console.log(data.time + '-' + this.channels[data.signalName].lastTime);
				if (this.channels[data.signalName].lastTime != 0 && Math.floor((data.time - this.channels[data
						.signalName].lastTime) / this.sampleTime) == 0) {
					// 采样时间不满足
					return;
				}
				// 当前数据集时间线是否超长，超长需要清除前面数据
				let timeLen = data.time - this.startTime;
				let localTime = data.time - this.originTime;
				if (timeLen > this.timeLineLength) {
					// 遍历channels对象，删除超过开始时间的数据，并更新开始时间
					for (let item in this.channels) {
						for (; this.channels[item].length != 0 && localTime - this.channels[item].list[0][0] > this
							.timeLineLength;) {
							this.channels[item].list.shift();
							this.channels[item].length--;
						}
					}
					this.startTime = data.time - this.timeLineLength;
				}
				// 更新采样数据到信号列表中
				this.channels[data.signalName].list.push([localTime, data.value]);
				this.channels[data.signalName].length++;
				this.channels[data.signalName].lastTime = data.time;
				if (data.color) this.channels[data.signalName].color = data.color;
				// console.log(JSON.stringify(this.channels));
				// 更新 option.series
				let series = [];
				for (let item in this.channels) {
					series.push({
						name: item,
						type: (this.channels[item].type === 'scatter') ? 'scatter' : 'line',
						showSymbol: false,
						smooth: true,
						color: this.channels[item].color,
						data: this.channels[item].list
					});
				}
				this.option.series = series;
			});
			// console.log(Math.floor(3/2));
			// setInterval(function() {
			// 	for (var i = 0; i < 5; i++) {
			// 		if (data.length > 100) data.shift();
			// 		data.push(randomData());
			// 	}
			// }, 1000);
		},
		methods: {}
	}
</script>

<style>
	.content {
		width: 98%;
		height: 98%;
		left: 0;
		top: 0;
		background-color: rgba(0, 122, 255, 0.1);
		position: absolute;
		border-radius: 20rpx;
		padding: 1% 1%;
		margin: 0 0;
		/* transform: rotate(90deg); */
		/* overflow: auto; */
	}

	.oscilloscope-container {
		width: calc(90vmax);
		height: calc(94vmin);
		/* left: 0%;
		top: 0; */
		/* background-color: rgba(0, 122, 255, 0.7); */
		position: absolute;
		border-radius: 20rpx;
		padding: 0% 0%;
		margin: 0 0;
		transform: rotate(90deg);
	}
</style>
