<template>
	<view>
		
		<wifi-link v-if="linkMethod==='udp'" :bleNamePrefix="settingFormData.blePrefix" @onSetting="onSetting" @onRecvString="onRecvString" @onBluetoothLinked="onBluetoothLinked"></wifi-link>
		<bluetooth-link v-if="linkMethod==='ble'" :bleNamePrefix="settingFormData.blePrefix" @onSetting="onSetting" @onBluetoothLinked="onBluetoothLinked" @onRecvString="onRecvString" @onBluetoothLosted="onBluetoothLosted"></bluetooth-link>
		<view>
			<uni-row class="device-operator-button">
				<uni-col :span="22" :offset="1" class="device-operator-col">
					<uni-row class="control-switch-row">
						<uni-col :span="8" class="">
							<uni-row class="safety-lock-switch">
								<switch class="edit-safety-switch" :checked="safetyLock" @change="safetyLockChange" />
								<p>保护锁</p>
							</uni-row>
						</uni-col>
						<uni-col :span="8" :offset="8" class="">
							<uni-row class="edit-mode-switch">
								<p>编辑模式</p>
								<switch class="edit-safety-switch" :checked="editMode" @change="editModeChange" />
							</uni-row>
						</uni-col>
					</uni-row>
					<uni-row v-for="group in (statusButtonList.length/4)" :key="'status'+group" class="status-button-row">
						<uni-fav v-for="(item, i) in statusButtonList.slice(4*(group-1), 4*group)" :key="item.id" 
						@click="statusButtonToggle(item)" 
						class="status-button-style" circle :star="false"
						:checked="statusButtonActive[item.id-1]" bgColorChecked="#007AFF" fgColorChecked="#FFFFFF"
						:class="statusButtonActive[item.id-1] ? 'status-button-checked' : 'status-button-unchecked'"
						:style="{opacity: (!editMode && item.hidden) ? 0 : 1}"
						:contentText="{contentDefault: item.name,contentFav: item.name}">
						</uni-fav>
					</uni-row>
					<uni-row class="status-button-row">
					</uni-row>
					<uni-row v-for="group in (normalButtonList.length/3)" :key="'narmal'+group" class="normal-button-row">
						<uni-fav v-for="(item, i) in normalButtonList.slice(3*(group-1), 3*group)" :key="item.id" @click="normalButtonToggle(item)" 
						@touchstart="onTouchstart(item)" @touchend="onTouchend(item)" class="normal-button-style" circle :star="false" 
						:style="{opacity: (!editMode && item.hidden) ? 0 : 1}"
						:contentText="{contentDefault: item.name,contentFav: item.name}" fgColor="#FFFFFF" fgColorChecked="#000000"></uni-fav>
					</uni-row>
					<view>
						<uni-popup ref="message" type="message">
							<uni-popup-message type="success" :message="TipMessageText" :duration="1000"></uni-popup-message>
						</uni-popup>
					</view>
					<view>
						<uni-popup ref="popup" background-color="#fff" @change="change">
							<view class="popup-content" >
								<uni-card title="按钮功能设置" :is-shadow="false">
									<uni-forms ref="baseForm" :modelValue="baseFormData">
										<uni-forms-item label="显示名称" required>
											<uni-easyinput v-model="baseFormData.name" placeholder="名称" />
										</uni-forms-item>
										<uni-forms-item label="触发命令">
											<uni-easyinput v-model="baseFormData.activeCmd" placeholder="命令" />
										</uni-forms-item>
										<uni-forms-item label="释放命令">
											<uni-easyinput v-model="baseFormData.inactiveCmd" placeholder="命令" />
										</uni-forms-item>
										<uni-forms-item label="回车换行">
											<uni-data-checkbox v-model="baseFormData.withCRLF" :localdata="boolSelect" />
										</uni-forms-item>
										<uni-forms-item label="是否隐藏">
											<uni-data-checkbox v-model="baseFormData.hidden"  :localdata="boolSelect" />
										</uni-forms-item>
										<button type="primary" @click="formCommit">确定</button>
									</uni-forms>
								</uni-card>
							</view>
						</uni-popup>
					</view>
					
					<view>
						<uni-popup ref="popupSetting" background-color="#fff" @change="change">
							<view class="popup-content" >
								<uni-card title="应用设置" :is-shadow="false">
									<uni-forms ref="settingForm" :modelValue="settingFormData">
										<uni-forms-item label="连接方式">
											<uni-data-checkbox v-model="linkMethod" :localdata="linkSelect" />
										</uni-forms-item>
										<uni-forms-item label="蓝牙前缀">
											<uni-easyinput v-model="settingFormData.blePrefix" placeholder="" />
										</uni-forms-item>
										<uni-forms-item label="设备版本">
											<uni-data-checkbox v-model="settingFormData.version" :localdata="versionSelect" />
										</uni-forms-item>
										<uni-forms-item label="使用角色">
											<uni-data-checkbox v-model="settingFormData.class" :localdata="classSelect" />
										</uni-forms-item>
										<!-- <uni-forms-item label="应用url">
											<uni-easyinput v-model="settingFormData.updateUrl" placeholder="" />
										</uni-forms-item> -->
										<uni-forms-item label="应用更新">
											<button type="primary" @click="applicationUpdate">更新</button>
										</uni-forms-item>
										<button type="primary" @click="settingCommit">确认设置</button>
									</uni-forms>
								</uni-card>
							</view>
						</uni-popup>
					</view>
					
				</uni-col>
			</uni-row>
		</view>
	</view>
</template>

<script>
	export default {
		name:"home",
		data() {
			return {
				bluetoothLinked: false,
				bluetoothSend: function(str) {},
				editMode: false,
				safetyLock: false,
				statusButtonActive: [false, false,false, false,false, false,false, false,false, false,false, false,false, false,false, false],
				statusButtonList: [],
				normalButtonList: [],
				baseFormData: {
					hidden: 0,
					withCRLF: 1
				},
				settingFormData: {
					blePrefix: 'SWIFT-',
					version: '2.0',
					class: 'develop', // develop produce delivery
					updateUrl: 'https://swift-helper.oss-cn-hangzhou.aliyuncs.com/swifthelper.json'
				},
				boolSelect: [{
						text: '是',
						value: 1
					}, {
						text: '否',
						value: 0
					}],
				linkSelect: [{
					text: 'BLE',
					value: 'ble'
				}, {
					text: 'UDP',
					value: 'udp'
				}],
				versionSelect: [{
					text: '1.2',
					value: '1.2'
				}, {
					text: '1.3',
					value: '1.3'
				}, {
					text: '1.4',
					value: '1.4'
				}, {
					text: '2.0',
					value: '2.0'
				}, {
					text: '其他',
					value: 'other'
				}],
				classSelect: [{
					text: '研发',
					value: 'develop'
				}, {
					text: '生产',
					value: 'produce'
				}, {
					text: '交付',
					value: 'delivery'
				}],
				editItem: {},
				TipMessageText: '',
				linkMethod: "ble"
			};
		},
		created() {
			let config = uni.getStorageSync('swiftHelperGlobalConfig');
			if (config.updateUrl) this.settingFormData.updateUrl = config.updateUrl;
			this.settingFormData.blePrefix = config.bleNamePrefix;
			let home = config.home;
			if (home) {
				this.statusButtonList = home.statusButtonList;
				this.normalButtonList = home.normalButtonList;
			}
		},
		beforeMount() {
			// 读取存储中的配置，没有则使用默认的值
			uni.$on('bluetoothSendString', str=>{
				console.log(str);
				if (this.bluetoothLinked) {
					this.bluetoothSend(str);
				}
			});
		},
		methods: {
			onRecvString(str) {
				// console.log("+++ " + str);
				let line = str;
				let match = line.match(/\{config:[a-z\_\.]{2,}=[0-9]{1,}\}/);
				if (match) {
					console.log();
					let key = match[0].split(':')[1].match(/[a-z\_\.]{2,}/)[0];
					let value = parseInt(match[0].match(/[0-9]{1,}/)[0]);
					console.log(key + "=" + value);
					uni.$emit('onRecvConfigData', {key: key, value: value});
				}
				match = line.match(/\{testInfo:.{1,}\}/);
				if (match) {
					// let string = match[0].split(':')[1].split('}')[0];
					let arr = match[0].split(':');
					arr.shift();
					let string = arr.join(":").split('}')[0];
					// console.log(string);
					uni.$emit('onRecvTestData', string);
				}
				uni.$emit('bluetoothRecv', str);
			},
			
			onBluetoothLinked(param) {
				this.bluetoothLinked = param.linked;
				this.bluetoothSend = param.send;
			},
			onBluetoothLosted(param) {
				this.bluetoothLinked = false;
				this.bluetoothSend = null;
			},
			change() {
				
			},
			
			onSetting() {
				let config = uni.getStorageSync('swiftHelperGlobalConfig');
				if (config) {
					this.settingFormData.blePrefix = config.bleNamePrefix;
					this.settingFormData.updateUrl = config.updateUrl;
				}
				this.$refs.popupSetting.open("bottom");
			},
			
			tipMessageToggle(msg) {
				//this.$refs.message.close();
				this.TipMessageText = msg;
				this.$refs.message.open();
			},
			statusButtonToggle(item) {
				console.log(item);
				if (this.editMode) {
					this.baseFormData = item;
					this.editItem = item;
					this.$refs.popup.open("bottom");
					return;
				}
				if (item.hidden || this.safetyLock) return ;
				if (this.bluetoothLinked != true) return;
				this.$set(this.statusButtonActive, item.id-1, !this.statusButtonActive[item.id-1]);
				let cmd = '';
				if (this.statusButtonActive[item.id-1]) {
					if (item.activeCmd.length == 0) return ;
					cmd = item.activeCmd + (item.withCRLF ? '\r\n' : '');
					this.tipMessageToggle(cmd);
				} else {
					if (item.inactiveCmd.length == 0) return ;
					cmd = item.inactiveCmd + (item.withCRLF ? '\r\n' : '');
					this.tipMessageToggle(cmd);
				}
				this.bluetoothSend(cmd);
			},
			normalButtonToggle(item) {
				if (this.editMode) {
					this.baseFormData = item;
					this.editItem = item;
					this.$refs.popup.open("bottom");
					return;
				}
				if (item.hidden || this.safetyLock) return ;
				if (this.bluetoothLinked != true) return;
			},
			safetyLockChange(e) {
				this.safetyLock = e.detail.value;
			},
			editModeChange(e) {
				this.editMode = e.detail.value;
				// 如果是结束编辑，将编辑后的数据写入到本地存储中，下一次打开应用会获取存储中的数据
			},
			onTouchstart(item) {
				if (item.hidden || !this.bluetoothLinked) return ;
				if (this.editMode) return ;
				if (this.safetyLock) return ;
				if (item.activeCmd.length == 0) return ;
				console.log(item);
				let cmd = item.activeCmd + (item.withCRLF ? '\r\n' : '');
				this.tipMessageToggle(cmd);
				this.bluetoothSend(cmd);
			},
			onTouchend(item) {
				if (item.hidden || !this.bluetoothLinked) return ;
				if (this.editMode) return ;
				if (this.safetyLock) return ;
				if (item.inactiveCmd.length == 0) return ;
				let cmd = item.inactiveCmd + (item.withCRLF ? '\r\n' : '');
				this.tipMessageToggle(cmd);
				this.bluetoothSend(cmd);
			},
			formCommit() {
				console.log(this.baseFormData);
				this.editItem = this.baseFormData;
				if (this.editItem.hidden) this.editItem.name = '';
				// store
				let config = uni.getStorageSync('swiftHelperGlobalConfig');
				if (config) {
					config.home.statusButtonList = this.statusButtonList;
					config.home.normalButtonList = this.normalButtonList;
					uni.setStorageSync('swiftHelperGlobalConfig', config);
				}
				
				this.$refs.popup.close();
			},
			settingCommit() {
				let config = uni.getStorageSync('swiftHelperGlobalConfig');
				if (config) {
					config.bleNamePrefix = this.settingFormData.blePrefix;
					config.updateUrl = this.settingFormData.updateUrl;
					uni.setStorageSync('swiftHelperGlobalConfig', config);
				}
				this.$refs.popupSetting.close();
			},
			applicationUpdate() {
				let targetUrl = `https://swift-helper.oss-cn-hangzhou.aliyuncs.com/swifthelper-${this.settingFormData.class}-${this.settingFormData.version}.json`;
				console.log(targetUrl);
				uni.request({
					url: targetUrl,
					success: res => {
						uni.setStorageSync('swiftHelperGlobalConfig', res.data);
						let config = uni.getStorageSync('swiftHelperGlobalConfig');
						this.statusButtonList = config.home.statusButtonList;
						this.normalButtonList = config.home.normalButtonList;
						this.settingFormData.updateUrl = config.updateUrl;
						this.settingFormData.blePrefix = config.bleNamePrefix;
						uni.$emit("applicationUpdated", '');
						uni.showToast({
							title: '更新成功'
						});
					},
					fail: err => {
						console.log(err);
						uni.showToast({
							title: '更新失败'
						});
					}
				})
			}
		}
	}
</script>

<style>
	.device-operator-row {
		/* margin-bottom: 20rpx; */
		/* margin-top: 20rpx; */
	}
	.device-operator-col {
		height: calc(100vh - 100rpx - 256rpx);
		/* background-color: #07C160; */
		border-radius: 20rpx;
	}
	.status-button-row,.control-switch-row {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
		margin-top: 50rpx;
		margin-bottom: 50rpx;
	}
	.control-switch-row {
		margin-top: 0rpx;
	}
	.status-button-checked {
		background-color: rgba(0, 122, 255, 1);
	}
	.status-button-unchecked {
		background-color: rgba(0, 122, 255, .1) !important;
	}
	.status-button-disable {
		border: rgba(0, 122, 255, 0.1) solid 2rpx !important;
	}
	.status-button-hidden {
		opacity: 0;
	}
	.normal-button-row {
		display: flex;
		flex-direction: row;
		justify-content: space-between;
		align-items: center;
		/* margin-top: 100rpx; */
		margin-bottom: 80rpx;
	}
	.edit-mode-switch {
		display: flex;
		flex-direction: row;
		justify-content: flex-start;
		align-items: center;
	}
	.safety-lock-switch {
		display: flex;
		flex-direction: row;
		justify-content: flex-end;
		align-items: center;
	}
	.status-button-style {
		border: #007AFF solid 2rpx;
		transform: scale(1.3);
		/* background-color: rgba(0, 122, 255, .1) !important; */
		margin-left: 20rpx;
		margin-right: 20rpx;
	}
	.normal-button-style {
		/* border: #007AFF solid 2rpx; */
		transform: scale(1.8);
		background-color: rgba(0, 122, 255, .7) !important;
		margin-left: 40rpx;
		margin-right: 40rpx;
	}
	.normal-button-style:active {
		background-color: rgba(0, 122, 255, 1) !important;
	}
	.edit-safety-switch {
		transform: scale(0.7);
	}
	.popup-content {
		width: 100%;
		/* position: absolute; */
	}
	
</style>
