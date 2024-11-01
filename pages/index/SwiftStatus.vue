<template>
	<view>
		<view class="uni-padding-wrap uni-common-mt">
			<uni-segmented-control :current="current" :values="items" :style-type="styleType"
				:active-color="activeColor" @clickItem="onClickItem" />
		</view>
		<view class="content">
			<view v-show="current === 0">
				<log-show :content="logArray"></log-show>
			</view>
			<view v-show="current === 1">
				<io-watch :value="inputIoValue" :bitWidth="24"></io-watch>
			</view>
			<view v-show="current === 2" class="content-text">
				<io-output></io-output>
			</view>
			<view v-if="current === 3">
				<oscilloscope samplingEvent='data_sampling_event'></oscilloscope>
			</view>
			<view v-if="current === 4" class="content-text">
			</view>
			<view v-if="current === 5" class="content-text">
				<test></test>
			</view>
		</view>
		<uni-popup ref="logModeSwitch" type="top" @change="logModePopupChange">
			<uni-card title="标题文字" thumbnail="" extra="额外信息" note="Tips">
				<template v-slot:title>
					<uni-list>
						<uni-list-item :show-switch="true" :switchChecked="modeSwitchChecked" @switchChange="modeSwitchChange" title="复盘模式"/>
					</uni-list>
				</template>
				<textarea v-show="modeSwitchChecked" v-model="replayLog" :maxlength="-1" style="height: 50vh;"
					placeholder="请粘贴需要复盘的日志内容,注意日志的格式必须符合当前应用支持的规则!"></textarea>
				<view v-show="modeSwitchChecked" class="fast-forward-mut">
					<b style="font-size: 16px;">速度比（0无效）</b>
					<uni-number-box :min="-16" :max="16" :value="numberBoxValue" :step="2" @change="numberBoxChange"></uni-number-box>
				</view>
				<button v-show="modeSwitchChecked" :disabled="!replayLog.length" @click="startReplay" type="primary">开始复盘</button>
			</uni-card>
		</uni-popup>
	</view>
</template>

<script>
	export default {
		components: {},
		data() {
			return {
				items: ['日志', '输入IO', '输出IO', '曲线'/*, '坐标系', '执行步骤'*/],
				current: 0,
				activeColor: '#007aff',
				styleType: 'button',
				logArray: [],
				inputIoValue: 0x00FFFFFF,
				modeSwitchChecked: false,
				navBarTapTouched: false,
				replayLog: '',
				replayLogArray: '',
				timer: null,
				currentTime: 0,
				fastForward: 1,
				numberBoxValue: 0,
				timeSyncCount: 0
			}
		},
		beforeCreate() {
			uni.$on('bluetoothRecv', str=>{
				if (!this.modeSwitchChecked) this.lineProcess(str);
			});
		},
		onNavigationBarButtonTap(e) {
			if (this.navBarTapTouched) {
				this.$refs.logModeSwitch.close();
				this.navBarTapTouched = false;
			} else {
				this.$refs.logModeSwitch.open('top');
				this.navBarTapTouched = true;
			}
		},
		methods: {
			onClickItem(e) {
				if (this.current !== e.currentIndex) {
					this.current = e.currentIndex
				}
			},
			modeSwitchChange() {
				this.modeSwitchChecked = !this.modeSwitchChecked;
				this.replayLog = '';
				this.logArray = [];
				if (this.timer != null) {
					clearInterval(this.timer);
					this.timer = null;
				}
			},
			numberBoxChange(e) {
				this.numberBoxValue = e;
				if (e == 0) this.fastForward = 1;
				else this.fastForward = e;
			},
			replayLogTextFocus() {
				uni.hideKeyboard();
			},
			logModePopupChange(e) {
				if (e.show == false) this.navBarTapTouched = false;
			},
			lineProcess(line) {
				// 获取输入IO
				// console.log(line);
				this.logArray.push(line);
				let match_flag = false;
				let time_match = line.match(/^\[[0-9]{1,}:[0-9]{1,}:[0-9]{1,}\.[0-9]{1,}\]/);
				let time_str = '';
				if (time_match && time_match.length > 0) {
					if (this.timeSyncCount == 0) {
						this.timeSyncCount ++;
						time_str = time_match[0].substr(1, time_match[0].length - 2);
						let time = time_str.split(':');
						let cur_time = new Date();
						if (cur_time.getHours() != parseInt(time[0]) || cur_time.getMinutes() != parseInt(time[1])) {
							let set_time_cmd = 'xp rtc set ' + (Math.round(cur_time.getTime() / 1000) + 1);
							uni.$emit('bluetoothSendString', set_time_cmd + '\r\n');
						}
					} else {
						this.timeSyncCount ++;
						this.timeSyncCount %= 10;
					}
				}
				let match = line.match(/\{io_input_state:[0-9]{1,}\}/);
				if (match) {
					// console.log(123);
					let io_value = parseInt(match[0].match(/[0-9]{1,}/));
					console.log(io_value);
					this.inputIoValue = io_value;
					//return ;
				}
				match = line.match(/\{oscilloscope@[0-9a-zA-z\_]{1,}:[0-9]{1,}/);
				if (match) {
					let match_key = match[0].match(/@[0-9a-zA-z\_]{1,}:/)[0];
					match_key = match_key.substring(1, match_key.length - 1);
					console.log(match_key);
					let match_value = parseInt(match[0].match(/:[0-9]{1,}/)[0].substr(1));
					uni.$emit('data_sampling_event', {signalName: match_key, type: 'line', time:new Date().getTime(), value: match_value});
				}
				//if (!match_flag) {
				//}
			},
			startReplay() {
				this.logArray = [];
				if (this.timer != null) {
					clearInterval(this.timer);
					this.timer = null;
				}
				this.$refs.logModeSwitch.close();
				this.navBarTapTouched = false;
				this.replayLogArray = this.replayLog.replace(/\n{2,}/g, '\n').replace(/\r/g, '')
						.split('\n').map(item=>{return item.trim()}).filter(item=>item.length!=0);
				// 设置一个定时器去顺序处理日志数组
				this.timer = setInterval(()=>{
					if (this.replayLogArray.length == 0) {
						clearInterval(this.timer);
						this.timer = null;
						return ;
					}
					if (this.currentTime != 0) {
						let timeStep = this.fastForward > 0 ? (96 * this.fastForward) : (96 / (0-this.fastForward));
						//console.log(timeStep);
						this.currentTime += timeStep;
					}
					
					let line = '';
					let timeStr = '';
					let logTime = 0;
					while (1) {
						if (this.replayLogArray.length == 0) {
							clearInterval(this.timer);
							this.timer = null;
							return ;
						}
						line = this.replayLogArray[0];
						timeStr = line.match(/^\[[0-9]{2}:[0-9]{2}:[0-9]{2}\.[0-9]{3}\]/);
						// logTime = Date.parse(new Date("2000-01-01 " + timeStr));
						if (!timeStr) {
							this.lineProcess(this.replayLogArray.shift());
						} else {
							// 提取时间转换为时间戳
							timeStr = timeStr[0].match(/[0-9]{2}:[0-9]{2}:[0-9]{2}\.[0-9]{3}/);
							logTime = Date.parse(new Date("2000-01-01 " + timeStr));
							// console.log(this.currentTime + "======" + logTime);
							if (this.currentTime == 0 || this.currentTime > logTime) {
								this.lineProcess(this.replayLogArray.shift());
								this.currentTime = logTime;
							} else {
								break;	// 时间不到，等
							}
						}
					}
				}, 96);
			}
		}
	}
</script>

<style lang="scss">

	.uni-common-mt {
		margin-top: 10rpx;
	}

	.uni-padding-wrap {
		// width: 750rpx;
		padding: 0px 30px;
	}
	
	.content {
		width: 94%;
		margin-left: 3%;
		display: flex;
		flex-direction: row;
		align-items: center;
		justify-content: center;
		height: 90%;
		bottom: 2%;
		position: absolute;
		// background-color: #007AFF;
		// overflow: auto;
	}
	.fast-forward-mut {
		// background-color: #07C160;
		display: flex;
		flex-direction: row;
		justify-content: space-around;
		margin-top: 20rpx;
		margin-bottom: 20rpx;
	}
</style>
