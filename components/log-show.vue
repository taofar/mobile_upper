<template>
	<view>
		<scroll-view class="log-content" :scroll-y="true"
		show-scrollbar :scroll-into-view="scroll_line" :scroll-top="scrollTop" @scroll="handScroll">
			<p id="head_line"></p>
			<p :class="line.match(/\[info\]/) ? 'log-item-style-info' : line.match(/\[debug\]/) ? 'log-item-style-debug' : line.match(/\[warn\]/) ? 'log-item-style-warn' : line.match(/\[error\]/) ? 'log-item-style-error' : 'log-item-style-other'" 
				:style="smallFont?'font-size:8px':'font-size:12px'" v-for="(line,index) in logList">{{line}}</p>
			<br/><br/>
			<p id='tail_line'></p>
		</scroll-view>
		<view class="log-operator">
			<uni-icons color="#FFF" v-for="(item,index) in logOperator" class="log-operator-item" :type="item.type" @click="logOperatorClick(item)"></uni-icons>
		</view>
	</view>
</template>

<script>
	export default {
		name:"log-show",
		data() {
			return {
				logList: [],
				container: {},
				scrollTop: 10000,
				scroll_line: '',
				smallFont: true,
				track: true,
				logOperator: [
					{id: 1, type: 'arrow-up'},
					{id: 2, type: 'minus'},
					{id: 3, type: 'arrow-down'},
					{id: 4, type: 'font'},
					{id: 5, type: 'compose'},
					{id: 6, type: 'trash'}
				]
			};
		},
		props: {
			content: {
				type: Array,
				default:()=>[]
			}
		},
		watch:{
			'content': function(param) {
				this.logList = param;
				if (this.logList.length > 5000) this.logList = this.logList.shift();
				if (this.track) this.scrollTop += 10;
			}
		},
		mounted() {
			this.logList = this.content;
			tail_line: 'tail_line';
		},
		updated() {
			tail_line: 'tail_line';
		},
		methods:{
			trigger(e) {
				// console.log(e);
				this.subcontent[e.index].active = true;
			},
			handScroll(e) {
				this.scroll_line = '';
			},
			logOperatorClick(item) {
				if (item.id == 1) {
					this.scroll_line = 'head_line';
					this.track = false;
				} else if (item.id == 2) {
					this.scroll_line = '';
					this.track = false;
				} else if (item.id == 3) {
					this.scroll_line = 'tail_line';
					this.track = true;
				} else if (item.id == 4) {
					this.smallFont = !this.smallFont;
				} else if (item.id == 5) {
					let copy = this.logList.join('\r\n');
					uni.setClipboardData({
						data:copy,
						success() {
							uni.showToast({
								title:"复制成功"
							})
						}
					});
				} else if (item.id == 6) {
					this.logList.splice(0, this.logList.length);
				}
			}
		}
	}
</script>

<style>
	.log-content {
		width: 98%;
		height: 98%;
		left: 0;
		top: 0;
		background-color: rgba(0, 122, 255, 0.1);
		position: absolute;
		border-radius: 20rpx;
		padding: 1% 1%;
		/* overflow: auto; */
	}
	.log-operator {
		display: flex;
		flex-direction: column;
		justify-content: space-around;
		align-items: center;
		width: 10%;
		height: 40%;
		top: 30%;
		right: 0;
		background-color: rgba(0, 122, 255, 1);
		position: absolute;
		border-radius: 20rpx;
		box-shadow: 0 1px 5px 2px rgba(0,0,0,0.3);
	}
	.log-operator-item {
		font-weight: 500;
		transform: scale(1.5);
	}
	.log-item-style-debug {
		color: #07C160;
		font-weight: 100;
	}
	.log-item-style-info {
		color: #000;
		font-weight: 700;
	}
	.log-item-style-warn {
		color: #FFD01E;
		font-weight: 700;
	}
	.log-item-style-error {
		color: #F00;
		font-weight: 700;
	}
	.log-item-style-other {
		color: #666666;
	}
</style>
