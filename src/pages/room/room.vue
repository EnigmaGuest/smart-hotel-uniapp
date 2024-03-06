<template>
	<view class="flex flex-direction">
		<image src="../../static/bg/bg-img.png" mode="widthFix" class="bg-image"></image>
		<ax-navbar image-url="../../static/bg/bg-img.png">
			<view slot="content" class="navbar-title">房间</view>
		</ax-navbar>
		<view class="page-container flex flex-direction">
			<view class="margin-top-xs"></view>
			<u-skeleton :rows="8" :loading="loading" title>
				<block v-if="!roomInfo.length">
					<view class="flex flex-direction align-center justify-center">
						<image src="../../static/guijishuaka.png" class="null-image"></image>
						<text class="margin-top font30">当前暂无订单，快去下单入住吧~</text>
					</view>
					<view class="op-ad flex flex-direction">
						<view class="op-ad__title">热门推荐</view>
						<ax-hotel-card v-for="(hotel, index) in hotelList" :hotel="hotel" :key="index"></ax-hotel-card>
					</view>
					<u-safe-bottom></u-safe-bottom>
				</block>
				<!-- 单个详情 -->
				<block v-else>
					<u-transition :show="showSingle" mode="fade-left">
						<view class="ax-card-32 flex flex-direction" v-show="showSingle" v-if="room">
							<view class="flex ax-card-32__item align-center" style="position: relative">
								<view class="flex align-center" v-if="roomInfo.length > 1" @click="showList">
									<u-icon name="arrow-left" color="#D59E32" size="20"></u-icon>
									<text class="font32 main-color">返回列表</text>
								</view>

								<view class="share-btn" v-if="false">
									<u-button open-type="share" size="mini" plain>
										<text class="font24">分享房间</text>
										<u-icon name="share-square" color="#D59E32" size="14"></u-icon>
									</u-button>
								</view>
							</view>
							<view class="flex ax-card-32__item">
								<image :src="room.roomCover" mode="aspectFill" class="room__image"></image>
								<view class="flex flex-direction margin-top-xs" style="width: 480rpx">
									<text class="font32">{{ room.hotelName }}</text>
									<text class="font24 info-color margin-top-xs">{{ room.hotelAddress }}</text>
								</view>
							</view>
							<view class="flex ax-card-32__item justify-around">
								<ax-lable title="酒店详情" icon="ax-icon-hotel" @click="onGoHotelInfo(room)"></ax-lable>
								<ax-lable title="联系店家" @click="onCallHotel(room)"></ax-lable>
								<ax-lable title="地图导航" icon="ax-icon-dizhi" @click="onGoHotelAddress(room)"></ax-lable>
							</view>
							<view class="flex ax-card-32__item justify-between align-center">
								<view class="font28">{{ room.roomName }}</view>
								<view class="flex align-center" hover-class="active--hover" @click="goRoomType(room.roomId)">
									<text class="font24 main-color margin-right-xs">房型详情</text>
									<u-icon name="arrow-right" color="#D59E32" size="12"></u-icon>
								</view>
							</view>
							<view class="flex ax-card-32__item flex-direction">
								<view class="room__info room__info__bg">
									<text class="room__info__key">房间楼层</text>
									<text class="room__info__value">{{ room.roomFloor }}楼</text>
								</view>
								<view class="room__info room__info__bg">
									<text class="room__info__key">房间号</text>
									<text class="room__info__value">{{ room.roomNo }}</text>
								</view>
								<view class="room__info room__info__bg" v-if="room.wifi">
									<text class="room__info__key">WIFI密码</text>
									<text class="room__info__value">{{ room.wifi }}</text>
								</view>
								<view class="room__info room__info__bg" v-if="room.openPassword">
									<text class="room__info__key">开门密码</text>
									<text class="room__info__value">{{ room.openPassword }}</text>
								</view>
								
							</view>
							<view class="flex ax-card-32__item align-center justify-around">
								<view class="flex flex-direction">
									<text class="font32">
										{{ getMMDD(room.startDate) }}
										<text class="font24">{{ getWeek(room.startDate) }}</text>
									</text>
									<text class="font24">{{ getTime(room.startDate) }}后</text>
								</view>
								<ax-time-capsule :text="`${room.days}晚`"></ax-time-capsule>
								<view class="flex flex-direction">
									<text class="font32">
										{{ getMMDD(room.endDate) }}
										<text class="font24">{{ getWeek(room.endDate) }}</text>
									</text>
									<text class="font24">{{ getTime(room.endDate) }}前</text>
								</view>
							</view>
						</view>
						<view class="ax-card-32 flex flex-direction" v-show="showSingle">
							<view class="font32 ax-card-32__item">客房服务</view>
							<view class="ax-card-32__item flex align-center justify-between">
								<view class="room__server-btn" @click="openDoor(room.openDoorMethod)">
									<image src="../../static/room/openDoor.png"></image>
									<text class="room__server-btn__text">一键开门</text>
								</view>
								<view class="room__server-btn" @click="continuationOrder(room.orderId)">
									<image src="../../static/room/continue.png"></image>
									<text class="room__server-btn__text">我要续住</text>
								</view>
								<view class="room__server-btn" @click="onCleanRoom(room.orderId, room.roomItemId)">
									<image src="../../static/room/clean.png"></image>
									<text class="room__server-btn__text">保洁打扫</text>
								</view>
								<view class="room__server-btn" @click="outRoom(room.orderId)">
									<image src="../../static/room/outRoom.png"></image>
									<text class="room__server-btn__text">离店退房</text>
								</view>
							</view>
						</view>
					</u-transition>
					<!-- 单个结束 -->
					<!-- 多个 -->
					<u-transition :show="!showSingle && roomInfo.length" mode="fade-right">
						<view class="ax-card-32 flex flex-direction" v-for="(item, index) in roomInfo" :key="index" v-if="!showSingle && roomInfo.length">
							<view class="flex align-center ax-card-32__item">
								<uni-icons type="ax-icon-hotel" custom-prefix="ax-icon" size="18" color="#000"></uni-icons>
								<text class="font32 margin-left-xxs">{{ item.hotelName }}</text>
							</view>
							<view class="flex ax-card-32__item">
								<image class="room__image" :src="item.roomCover"></image>
								<view class="flex flex-direction margin-top-xs">
									<text class="font32">{{ item.roomName }}</text>
									<text class="font24 info-color margin-top-xs">
										房间号：
										<text class="font-bold black-color">{{ item.roomNo }}</text>
									</text>
								</view>
							</view>
							<view class="ax-card-32__item flex justify-end">
								<ax-button content="一键开门" class="margin-right-sm" @click="openDoor(item.openDoorMethod)"></ax-button>
								<ax-button content="查看详情" @click="showRoomSingle(item)"></ax-button>
							</view>
						</view>
					</u-transition>
				</block>
			</u-skeleton>
			<view class="margin-bottom"></view>
		</view>
	</view>
</template>

<script>
const dayjs = require('@/utils/dayjs.min.js');
let smartLock = requirePlugin('smartLock');
export default {
	data() {
		return {
			roomInfo: [],
			showSingle: false,
			loading: true,
			hotelList: null,
			localtion: {},
			room: null,
			openLockObj: {}
		};
	},
	methods: {
		getMMDD(time) {
			return dayjs(time).format('MM月DD日');
		},
		getWeek(time) {
			let week = dayjs(time).format('d');
			switch (week) {
				case '0':
					return '周日';
				case '1':
					return '周一';
				case '2':
					return '周二';
				case '3':
					return '周三';
				case '4':
					return '周四';
				case '5':
					return '周五';
				case '6':
					return '周六';
			}
		},
		getTime(time) {
			return dayjs(time).format('HH:mm');
		},
		showRoomSingle(item) {
			this.showSingle = true;
			this.room = item;
		},
		showList() {
			this.showSingle = false;
		},
		continuationOrder(id) {
			uni.navigateTo({
				url: '/pages/order/continuation?id=' + id,
				success: (res) => {},
				fail: () => {},
				complete: () => {}
			});
		},
		playSuccess() {
			const innerAudioContext = uni.createInnerAudioContext();
			innerAudioContext.autoplay = true;
			innerAudioContext.src = '/static/mp3/open_room_success.mp3';
			innerAudioContext.onPlay(() => {
				console.log('开始播放');
			});
		},
		playError() {
			const innerAudioContext = uni.createInnerAudioContext();
			innerAudioContext.autoplay = true;
			innerAudioContext.src = '/static/mp3/open_room_error.mp3';
			innerAudioContext.onPlay(() => {
				console.log('开始播放');
			});
		},
		async openDoor(obj) {
			uni.showLoading({
				title: '一键开锁中',
				mask: false
			});
			if (obj.doorLockType == 'YY') {
				smartLock.openCloseBleLock(obj.deviceCode, 1, (res) => {
					console.log(res);
					uni.hideLoading();
					if (res.data.result == 0) {
						this.playSuccess();
						uni.showToast({
							title: '开门成功'
						});
					} else {
						this.playError();
						uni.showModal({
							title: '开门失败',
							content: '请靠近房门,同时确保手机网络通畅并打开手机蓝牙和定位服务(GPS),或者使用密码开门',
							showCancel: false,
							cancelText: '',
							confirmText: '确定',
							success: (res) => {},
							fail: () => {},
							complete: () => {}
						});
					}
				});
			} else {
				this.playError();
			}
		},
		dianCan() {
			uni.showToast({
				title: '该酒店暂不支持点餐',
				icon: 'none'
			});
		},
		async onCleanRoom(orderId, roomItemId) {
			uni.showModal({
				title: '房间打扫',
				content: '是否需要酒店打扫该房间？',
				showCancel: true,
				cancelText: '查看记录',
				confirmText: '确定打扫',
				success: (res) => {
					if (res.confirm) {
						this.postClean(orderId, roomItemId);
					}else if (res.cancel){
						uni.navigateTo({
							url: '/pages/room/clear'
						});
					}
				},
				fail: () => {},
				complete: () => {}
			});
		},
		async postClean(orderId, roomItemId) {
			let qes = await this.$api.postCleanRoom({ orderId: orderId, roomItemId: roomItemId });
			if (qes.code == 0) {
				uni.navigateTo({
					url: '/pages/room/clear'
				});
			}
		},
		async getHomeHotel() {
			let opstion = this.$st.getIndexPageData();
			if (opstion != null && !this.hotelList) {
				this.localtion = JSON.parse(opstion.localtion);
				let data = await this.$api.getHotel({ longitude: this.localtion.longitude, latitude: this.localtion.latitude, recommendSize: 10 });
				this.hotelList = data.data.records;
				console.log(this.hotelList);
			}
		},
		goRoomType(id) {
			uni.navigateTo({
				url: '/pages/hotel/room/room?id=' + id
			});
		},
		async outRoom(id) {
			uni.showModal({
				title: '提示',
				content: '是否取消订单,取消后无法享受免办理续住',
				success: async (r) => {
					if (r.confirm) {
						let res = await this.$api.checkOutRoom(id);
						if (res.code === 0) {
							uni.showToast({
								title: '退房成功',
								icon: 'success'
							});
							let ress = await this.$api.getBindRoomList();
							if (ress.code === 0) {
								this.roomInfo = ress.data;
								if (!ress.data.length) {
									if (!this.hotelList) {
										await this.getHomeHotel();
									}
								}
								if (ress.data.length === 1) {
									this.showSingle = true;
									this.room = ress.data[0];
								}
							}
						}
					}
				}
			});
		},
		async getData() {
			let res = await this.$api.getBindRoomList();
			if (res.code === 0) {
				this.roomInfo = res.data;
				if (!res.data.length) {
					if (!this.hotelList) {
						await this.getHomeHotel();
					}
				}
				if (res.data.length === 1) {
					this.showSingle = true;
					this.room = res.data[0];
				}
			}
			this.loading = false;
		},
		initSmartLockPlus() {
			//initSmartLock 中需要5个参数，分别为：appid，appkey，二次开发者登陆dms的账号、二次开发者登陆dms的密码，初始化回调函数。前四个参数请微信联系18988768327或者13530382506获取
			smartLock.initSmartLock('appid', 'appkey', '****', '*****', (respones) => {
				if (respones.data.result != 0) {
					uni.showToast({
						title: '初始化失败',
						icon: 'none'
					});
				}
			});
		},
		onGoHotelInfo(item) {
			uni.navigateTo({
				url: '/pages/hotel/info/info?btn=false&id=' + item.hotelId
			});
		},
		onCallHotel(item){
			if(item.hotelPhone ){
				uni.makePhoneCall({
					phoneNumber: item.hotelPhone 
				});
			}else{
				uni.showModal({
					title: '提示',
					content: '未能获取到该商家的联系方式',
					showCancel: false,
					cancelText: '',
					confirmText: '确定',
					success: res => {},
					fail: () => {},
					complete: () => {}
				});
			}
		},
		onGoHotelAddress(item){
			uni.openLocation({
				latitude: item.latitude,
				longitude: item.longitude,
				name: item.hotelName,
				address: item.hotelAddress,
				success: () => {}
			});
		}
	},
	async onShow() {
		this.getData();
		this.initSmartLockPlus();
	},
	onShareAppMessage(res) {
		if (res.from === 'button') {
			// 来自页面内分享按钮
			console.log(res.target);
		}
		// return {
		// 	title: '分享给您了一个房间',
		// 	path: '/pages/room/bind?id=123'
		// };
	}
};
</script>
<style lang="scss">
page {
	background-color: $ax-bg-color;
}
</style>
<style lang="scss" scoped>
.bg-image {
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	width: 100%;
	z-index: 1;
}

.navbar-title {
	padding: 0 32rpx;
	width: 100%;
	font-size: 40rpx;
	font-weight: bold;
	color: $ax-text-color-black;
}

.page-container {
	z-index: 2;
	padding: 0 32rpx;
}
.share-btn {
	position: absolute;
	top: 0rpx;
	right: 0rpx;
	width: 160rpx;
	height: 48rpx;
	display: flex;
	text-align: center;
	align-items: center;
	justify-content: center;
	border-radius: 10rpx 10rpx 10rpx 10rpx;
	font-size: 24rpx;
	color: #d59e32;
}

.room {
	&__server-btn {
		display: flex;
		flex-direction: column;
		align-items: center;

		& image {
			width: 48rpx;
			height: 48rpx;
			// background-color: #f8f8f8;
			border-radius: 0rpx 0rpx 0rpx 0rpx;
			margin-bottom: 16rpx;
		}

		&__text {
			font-size: 24rpx;
			color: #242424;
		}
	}

	&__image {
		width: 120rpx;
		height: 120rpx;
		background-color: #f8f8f8;
		border-radius: 10rpx 10rpx 10rpx 10rpx;
		margin-right: 16rpx;
	}

	&__info {
		&__key {
			font-size: 26rpx;
			font-weight: normal;
			color: #999999;
		}

		&__value {
			font-size: 40rpx;
			font-weight: 500;
			color: #d59e32;
		}

		&__bg {
			height: 86rpx;
			background: #f8f8f8;
			border-radius: 16rpx 16rpx 16rpx 16rpx;
			display: flex;
			align-items: center;
			justify-content: space-between;
			padding: 0 32rpx;

			&:not(:last-child) {
				margin-bottom: 16rpx;
			}
		}
	}
}
.null-image {
	width: 750rpx;
	height: 440rpx;
	border-radius: 0rpx 0rpx 0rpx 0rpx;
}
.op-ad {
	&__title {
		font-size: 36rpx;
		color: $ax-text-color-black;
		font-weight: normal;
		margin: 24rpx 0;
	}

	&__card {
		&:not(:first-child) {
			margin-top: 24rpx;
		}
	}
}
</style>
