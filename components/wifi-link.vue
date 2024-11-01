<template>
	<view class="content">
		<uni-row class="device-link-row">
			<uni-col :span="22" :offset="1">
				<view class="device-link-col" :class="{'dev-linked': bluetoothLinked}">
					<uni-row class="">
						<uni-col :span="14">
							<view class="bluetooth-info">
								<h2>{{linkInfo.name}}</h2>
								<h3>{{linkInfo.mac}}</h3>
							</view>
						</uni-col>
						<uni-col :span="5">
							<view class="bluetooth-button" v-on:click="onRefreshButtonClicked()">
								<uni-icons :type="bluetoothLinked? 'close' : 'refresh-filled'" :class="{'bluetooth-linking': bluetoothLinking}" :color="bluetoothLinked ? '#007AFF' : '#FF5A5F'"
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
	import { UDPClient, UDPServer } from "@/uni_modules/uts-udp"
	export default {
		data() {
			return {
				bluetoothLinked: false,
				bluetoothLinking: false,
				linkInfo: {
					name: '', //'1km-140',
					mac: '未连接设备', //'12:23:34:45:56:67',
					dbm: 0, //-50,
					service: [],
					characteristic: ''
				},
				adapterState: {},
				deviceInfo: {},
				deviceSearched: false,
				server: ""
			}
		},
		props: {
			bleNamePrefix: {
				type: String,
				default:()=>"SL202"
			}
		},
		beforeMount() {
			
		},
		methods: {
			onUdpMessage(res) {
				console.log(res);
				this.$emit('onRecvString', res);
			},
			onUdpError(err) {
				console.log(err);
			},
			bluetoothSendString(str) {
				let packet = {
					cmd: 3,	// publish
					topic: "",
					value: str
				};
				if (this.server) this.server.send(JSON.stringify(packet));
			},
			onRefreshButtonClicked(e) {
				if (this.bluetoothLinked == true) {
					// close TODO
					if (this.server) this.server.stop();
					this.bluetoothLinked = false;
					this.linkInfo.name = '';
					this.linkInfo.mac = '未连接设备';
					return ;
				}
				let self = this;
				// console.log(UDPClient.recv);
				UDPClient.send({
						host: '255.255.255.255',
						port: 9988,
						receiveTimeout: 5000,
						msg: JSON.stringify({
								cmd: 0,		// find and get hostname
								topic: "",
								value: ""
						}),
						enableRecive: true,
						onceReceive(data) {
							let recv_msg = JSON.parse(data.msg);
							console.log(!recv_msg.value.match(self.bleNamePrefix));
							if (!recv_msg.value.match(self.bleNamePrefix)) return;
							self.linkInfo.name = recv_msg.value;
							self.linkInfo.mac = data.host;
							self.bluetoothLinking = false;
							self.bluetoothLinked = true;
							
							const server = new UDPServer(54345, 4096);
							self.server = {
								listener: (recv, error) => {
									server.listener(recv, error);
								},
								send: (msg) => {
									server.send(msg, self.linkInfo.mac, 9988);
								},
								stop: () => {
									let packet = {
										cmd: 2,		// unsubscribe
										topic: "",
										value: ""
									};
									server.send(JSON.stringify(packet), self.linkInfo.mac, 9988);
									server.stop();
								}
							}
							self.server.listener(
								data => self.onUdpMessage(JSON.parse(data.msg).value),
								err => self.server.stop()
							);
							let packet = {
								cmd: 1,	// subscribe
								topic: "*",
								value: ""
							};
							console.log(JSON.stringify(packet));
							self.server.send(JSON.stringify(packet));
							self.$emit('onBluetoothLinked', {linked: true, send: self.bluetoothSendString});
							// 匹配主机名			{"hostname": "", "ip": ""}
						},
						onError(error) {
							this.bluetoothLinked = false;
							this.linkInfo.name = '';
							this.linkInfo.mac = '未连接设备';
							console.error(error);
						},
						onceReceiveTimeout() {
							console.warn('服务器超时未回复');
						},
					});
				// console.log("refreshButtonClicked");
				// // connect
				// client.send({data: "12345", port: 8889});
				this.bluetoothLinking = true;
				setTimeout(()=>{
					if (this.bluetoothLinking) {
						// close
						this.bluetoothLinking = false;
						this.server && this.server.stop();
					}
				}, 5000)
			},
			onSettingButtonClicked(e) {
				console.log("settingButtonClicked");
				this.$emit('onSetting', '');
			},

			showToastString(str_info) {
				return ;
				uni.showToast({
					title: str_info
				});
			},

		}
	};
	
	function toast(content, showCancel = false) {
		uni.showToast({
			title: content
		});
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
	
	.bluetooth-linking {
		animation: spin .5s linear infinite;
	}
	@keyframes spin{
		0% {
			transform: rotate(0deg);
		}
		100% {
			transform: rotate(360deg);
		}
	}
</style>
