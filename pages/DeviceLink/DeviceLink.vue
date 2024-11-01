<template>
	<view class="content">
		<uni-row class="device-link-row">
			<uni-col :span="22" :offset="1">
				<view class="device-link-col" :class="{'dev-linked': bluetoothLinked}">
					<uni-row class="">
						<uni-col :span="14">
							<view class="bluetooth-info">
								<h1>{{linkInfo.name}}</h1>
								<h3>{{linkInfo.mac}}</h3>
								<h3><b>{{linkInfo.dbm}} dBm</b></h3>
							</view>
						</uni-col>
						<uni-col :span="5">
							<view class="bluetooth-button" v-on:click="onRefreshButtonClicked()">
								<uni-icons type="refresh-filled" :color="bluetoothLinked ? '#007AFF' : '#FF5A5F'"
									size="60"></uni-icons>
							</view>
						</uni-col>
						<uni-col :span="5">
							<view class="bluetooth-button" v-on:click="onSettingButtonClicked()">
								<uni-icons type="gear-filled" :color="bluetoothLinked ? '#007AFF' : '#FF5A5F'"
									size="60"></uni-icons>
							</view>
						</uni-col>
					</uni-row>
				</view>
			</uni-col>
		</uni-row>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				title: '点击连接蓝牙设备',
				bluetoothLinked: false,
				linkInfo: {
					name: '', //'1km-140',
					mac: '未连接设备', //'12:23:34:45:56:67',
					dbm: 0, //-50,
					service: [],
					characteristic: ''
				},
				adapterState: {},
				deviceInfo: {},
				deviceSearched: false
			}
		},
		onLoad() {
			this.onBLEConnectionStateChange();
		},
		methods: {
			bluetoothSendString(str) {
				this.writeBLECharacteristicValue(str);
			},
			bluetoothRecvStringCallback(str) {
				console.log(str);
			},
			onRefreshButtonClicked(e) {
				console.log("refreshButtonClicked");
				this.closeBLEConnection();
				this.closeBluetoothAdapter();
				this.autoConnectBluetooth();
			},
			onSettingButtonClicked(e) {
				console.log("settingButtonClicked");
				uni.stopBluetoothDevicesDiscovery();
				this.closeBLEConnection();
			},

			showToastString(str_info) {
				uni.showToast({
					title: str_info
				});
			},

			autoConnectBluetooth() {
				// 蓝牙初始化
				uni.openBluetoothAdapter({
					success: res => {
						// 查询设备适配器状态
						uni.getBluetoothAdapterState({
							success: res => {
								this.adapterState = res;
								// 开始查询设备
								this.startBluetoothDevicesDiscovery();
							},
							fail: e => {
								console.log('获取本机蓝牙适配器状态失败，错误码：' + e.errCode);
								this.showToastString('获取蓝牙适配器状态失败, 错误码:' + e.errCode)
								if (e.errCode !== 0) {
									bluetoothLinkFailTip(e.errCode);
								}
							}
						});
					},
					fail: err => {
						this.showToastString("蓝牙初始化失败，请检查蓝牙是否打开");
					}
				})
			},
			// 搜索蓝牙设备
			getBluetoothDevices() {
				uni.getBluetoothDevices({
					success: res => {
						//console.log('获取蓝牙设备:' + JSON.stringify(res.devices));
						// console.log(JSON.stringify(res))
						if (this.deviceSearched) return;
						// 判断蓝牙设备中是否有 1km- 开头的设备，有则选中该设备直接连接
						for (let item of res.devices) {
							//if (item.name && item.name.length != 0) console.log(JSON.stringify(item));
							if (item.name.length > 4 && item.name.match(/^1KM-/)) {
								console.log(JSON.stringify(item));
								uni.stopBluetoothDevicesDiscovery();
								this.deviceSearched = true;
								this.deviceInfo = item;
								this.linkInfo.name = item.name;
								this.linkInfo.mac = item.deviceId;
								this.linkInfo.dbm = item.RSSI;
								this.linkInfo.service = item.advertisServiceUUIDs;
								//this.bluetoothLinked = true;
								this.createBLEConnection();
							}
						}
					},
					fail: e => {
						//console.log('获取蓝牙设备错误，错误码：' + e.errCode);
						this.showToastString('获取蓝牙设备错误，错误码：' + e.errCode);
						if (e.errCode !== 0) {
							bluetoothLinkFailTip(e.errCode);
						}
					}
				});
			},
			onBluetoothDeviceFound() {
				uni.onBluetoothDeviceFound(devices => {
					console.log('开始监听寻找到新设备的事件');
					if (this.deviceSearched) {
						uni.stopBluetoothDevicesDiscovery();
						return false;
					}
					this.getBluetoothDevices();
				});
			},
			startBluetoothDevicesDiscovery() {
				uni.startBluetoothDevicesDiscovery({
					success: e => {
						console.log('开始搜索蓝牙设备:' + e.errMsg);
						this.onBluetoothDeviceFound();
					},
					fail: e => {
						//console.log('搜索蓝牙设备失败，错误码：' + e.errCode);
						this.showToastString('搜索蓝牙设备失败，错误码：' + e.errCode);
						if (e.errCode !== 0) {
							bluetoothLinkFailTip(e.errCode);
						}
					}
				});
			},
			// 连接蓝牙设备
			createBLEConnection() {
				let deviceId = this.linkInfo.mac;
				uni.createBLEConnection({
					// 这里的 deviceId 需要已经通过 createBLEConnection 与对应设备建立链接
					deviceId,
					success: res => {
						//console.log(res);
						console.log('连接蓝牙成功:' + res.errMsg);
						// 连接设备后断开搜索 并且不能搜索设备
						//this.stopBluetoothDevicesDiscovery();
						uni.showToast({
							title: '连接成功',
							icon: 'success',
							duration: 2000
						});
						setTimeout(()=>{
							this.getBLEDeviceCharacteristics();
						}, 1000);
						//this.bluetoothLinked = true;
					},
					fail: e => {
						this.showToastString('连接低功耗蓝牙失败，错误码：' + e.errCode);
						if (e.errCode !== 0) {
							bluetoothLinkFailTip(e.errCode);
						}
					}
				});
			},
			// 断开蓝牙
			closeBLEConnection() {
				let deviceId = this.linkInfo.mac;
				uni.closeBLEConnection({
					deviceId,
					success: res => {
						this.bluetoothLinked = false;
						this.linkInfo = {
							name: '',
							mac: '未连接设备',
							dbm: 0,
							service: [],
							characteristic: []
						},
						this.adapterState = {};
						this.devicesList = [];
						this.deviceSearched = false;
						this.showToastString('当前蓝牙连接已经断开');
					},
					fail: e => {
						console.log('断开低功耗蓝牙失败，错误码：' + e.errCode);
						if (e.errCode !== 0) {
							bluetoothLinkFailTip(e.errCode);
						}
					}
				});
			},
			
			onBLEConnectionStateChange() {
				uni.onBLEConnectionStateChange(res => {
					// 该方法回调中可以用于处理连接意外断开等异常情况
					console.log(`蓝牙连接状态 -------------------------->`);
					console.log(JSON.stringify(res));
					if (!res.connected) {
						if (this.isStop) return;
						console.log('断开低功耗蓝牙');
						this.bluetoothLinked = false;
						this.linkInfo = {
							name: '',
							mac: '未连接设备',
							dbm: 0,
							service: [],
							characteristic: []
						},
						this.adapterState = {};
						this.devicesList = [];
						this.deviceSearched = false;
						this.showToastString('当前蓝牙连接已经断开');
					}
				});
			},
			
			closeBluetoothAdapter() {
				uni.closeBluetoothAdapter({
					success: res => {
						console.log('断开蓝牙模块成功');
					}
				});
			},
			
			// 读取特征值
			// uni.readBLECharacteristicValue
			// 获取服务
			// uni.getBLEDeviceServices
			// 获取特征值UUID
			getBLEDeviceCharacteristics() {
				let deviceId = this.linkInfo.mac;
				let serviceId = this.linkInfo.service[0];
				console.log(deviceId);
				console.log(serviceId);
				uni.getBLEDeviceCharacteristics({
					deviceId,
					serviceId,
					success: res => {
						console.log(JSON.stringify(res));
						console.log('获取特征值成功:' + res.errMsg);
						this.linkInfo.characteristic = res.characteristics;
						if (res.characteristics.length <= 0) {
							bluetoothLinkFailTip('获取特征值失败，请重试!');
							return;
						}
						for (let item of res.characteristics) {
							if (item.properties.notify) {
								console.log(item);
								this.notifyBLECharacteristicValueChange(item);
							}
						}
						this.bluetoothLinked = true;
					},
					fail: e => {
						console.log('获取特征值失败，错误码：' + e.errCode);
						if (e.errCode !== 0) {
							bluetoothLinkFailTip(e.errCode);
						}
					}
				});
			},
			// 建立发送接收方法
			stringToArrayBuffer(str) {
				var bytes = new Array();
				var len, c;
				len = str.length;
				for (var i = 0; i < len; i++) {
					c = str.charCodeAt(i);
					if (c >= 0x010000 && c <= 0x10FFFF) {
						bytes.push(((c >> 18) & 0x07) | 0xF0);
						bytes.push(((c >> 12) & 0x3F) | 0x80);
						bytes.push(((c >> 6) & 0x3F) | 0x80);
						bytes.push((c & 0x3F) | 0x80);
					} else if (c >= 0x000800 && c <= 0x00FFFF) {
						bytes.push(((c >> 12) & 0x0F) | 0xE0);
						bytes.push(((c >> 6) & 0x3F) | 0x80);
						bytes.push((c & 0x3F) | 0x80);
					} else if (c >= 0x000080 && c <= 0x0007FF) {
						bytes.push(((c >> 6) & 0x1F) | 0xC0);
						bytes.push((c & 0x3F) | 0x80);
					} else {
						bytes.push(c & 0xFF);
					}
				}
				var array = new Int8Array(bytes.length);
				for (var i = 0; i <= bytes.length; i++) {
					array[i] = bytes[i];
				}
				return array.buffer;
			},
			
			writeBLECharacteristicValue(str) {
				let deviceId = this.linkInfo.mac;
				let serviceId = this.linkInfo.service[0];
				let characteristicId = '';
				for (let item of this.linkInfo.characteristic) {
					if (item.properties.write) {
						characteristicId = item.uuid;
					}
				}
				if (characteristicId.length <= 0) {
					console.log("没找到可发送的特征值");
					return ;
				}
				uni.writeBLECharacteristicValue({
					deviceId,
					serviceId,
					characteristicId,
					writeType: "writeNoResponse",
					value: this.stringToArrayBuffer(str),
						success: res => {
							//console.log(JSON.stringify(res));
							//this.notifyBLECharacteristicValueChange();
						},
						fail(e) {
							console.log('写设备数据值失败，错误码：' + e.errCode);
							if (e.errCode !== 0) {
								bluetoothLinkFailTip(e.errCode);
							}
						}
				});
			},
			
			// 特征值改变后通知读取变化后的值
			notifyBLECharacteristicValueChange(param) {
				let deviceId = this.linkInfo.mac;
				let serviceId = this.linkInfo.service[0];
				let characteristicId = param.uuid;
				let notify = param.properties.notify;
				console.log(deviceId);
				console.log(serviceId);
				console.log(characteristicId);
				console.log(notify);
				let callback = this.onBLECharacteristicValueChange;
				uni.notifyBLECharacteristicValueChange({
					state: notify, // 启用 notify 功能
					// 这里的 deviceId 需要已经通过 createBLEConnection 与对应设备建立链接
					deviceId,
					// 这里的 serviceId 需要在 getBLEDeviceServices 接口中获取
					serviceId,
					// 这里的 characteristicId 需要在 getBLEDeviceCharacteristics 接口中获取
					characteristicId,
					success(res) {
						console.log('notifyBLECharacteristicValueChange success:' + res.errMsg);
						console.log(JSON.stringify(res));
						//this.onBLECharacteristicValueChange();
						callback();
					},
					fail(err) {
						console.log(err);
					}
				});
			},
			onBLECharacteristicValueChange() {
				// 必须在这里的回调才能获取
				uni.onBLECharacteristicValueChange(characteristic => {
					//console.log('监听低功耗蓝牙设备的特征值变化事件成功');
					console.log(String.fromCharCode.apply(null, new Uint8Array(characteristic.value)));
					//this.bluetoothSendString(String.fromCharCode.apply(null, new Uint8Array(characteristic.value)));
					if (this.bluetoothRecvStringCallback) {
						this.bluetoothRecvStringCallback(String.fromCharCode.apply(null, new Uint8Array(characteristic.value)));
					}
				});
			}
		}
	};
	
	function toast(content, showCancel = false) {
		uni.showToast({
			title: content
		});
	}
	
	function bluetoothLinkFailTip(code, errMsg) {
		switch (code) {
			case 10000:
				toast('未初始化蓝牙适配器');
				break;
			case 10001:
				toast('未检测到蓝牙，请打开蓝牙重试！');
				break;
			case 10002:
				toast('没有找到指定设备');
				break;
			case 10003:
				toast('连接失败');
				break;
			case 10004:
				toast('没有找到指定服务');
				break;
			case 10005:
				toast('没有找到指定特征值');
				break;
			case 10006:
				toast('当前连接已断开');
				break;
			case 10007:
				toast('当前特征值不支持此操作');
				break;
			case 10008:
				toast('其余所有系统上报的异常');
				break;
			case 10009:
				toast('Android 系统特有，系统版本低于 4.3 不支持 BLE');
				break;
			default:
				toast(errMsg);
		}
	}
</script>

<style>
	.device-link-row {
		margin-bottom: 40rpx;
		margin-top: 40rpx;
	}

	.device-link-col {
		height: 256rpx;
		border-radius: 10px;
		background-color: rgba(255, 90, 95, 0.2);
	}

	.dev-linked {
		background-color: #007AFF;
		background-color: rgba(0, 122, 255, 0.2);
	}

	.bluetooth-info {
		display: flex;
		flex-direction: column;
		justify-content: center;
		height: 256rpx;
		padding-left: 30rpx;
		margin-top: -5px;
	}

	.bluetooth-button {
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-self: center;
		height: 256rpx;
	}
</style>
