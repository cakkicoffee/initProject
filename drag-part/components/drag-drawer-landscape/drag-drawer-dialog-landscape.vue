<template>
	<drag-drawer-landscape
		:drawerId="drawerId"
		:mask="mask"
		ref="dragDrawerLandscape"
		@closed="closed"
	>
		<view class="drag-drawer-dialog-landscape">
			<view class="drag-drawer-dialog-landscape__content">
				<slot>
					<view
						v-if="imgUrl"
						class="drag-drawer-dialog-landscape__picture"
						:style="{ backgroundImage: 'url(' + imgUrl + ')' }"
					></view>
					<view class="drag-drawer-dialog-landscape__title">{{ title }}</view>
					<text v-if="message" class="drag-drawer-dialog-landscape__message">
						{{ message }}
					</text>
				</slot>
			</view>
			<view class="drag-drawer-dialog-landscape__foot">
				<view
					v-if="showConfirmButton"
					class="drag-drawer-dialog-landscape__button drag-drawer-dialog-landscape__confirm"
					:style="confirmButtonStyle"
					@tap="confirmClick"
				>
					{{ confirmText }}
				</view>
				<view
					class="drag-drawer-dialog-landscape__button"
					:style="cancelButtonStyle"
					@tap="cancelClick"
				>
					{{ cancelText }}
				</view>
			</view>
		</view>
	</drag-drawer-landscape>
</template>

<script>
const buleStyle = {
	backgroundColor: '#0084FF',
	color: '#ffffff'
}
const whiteStyle = {
	backgroundColor: '#ffffff',
	color: '#2B2F33',
	border: 100 / 812 + 'vw solid #CCD2D9'
}

import DragDrawerLandscape from './drag-drawer-landscape'
export default {
	name: 'DragDrawerDialogLandscape',
	components: {
		DragDrawerLandscape
	},
	props: {
		drawerId: {
			type: String,
			default: 'drag_drawer_dialog_landscape_' + new Date().getTime()
		}
	},
	computed: {
		confirmButtonStyle() {
			let style = this.confirmStyle
			if (this.confirmStyleModel) {
				if (this.confirmStyleModel == 'blue') {
					style = buleStyle
				}
				if (this.confirmStyleModel == 'white') {
					style = whiteStyle
				}
			}
			return style
		},
		cancelButtonStyle() {
			let style = this.cancelStyle
			if (this.cancelStyleModel) {
				if (this.cancelStyleModel == 'blue') {
					style = buleStyle
				}
				if (this.cancelStyleModel == 'white') {
					style = whiteStyle
				}
			}
			return style
		}
	},
	data() {
		return {
			imgUrl: '',
			title: '',
			message: '',
			mask: true,
			showConfirmButton: true,
			confirmText: '确认',
			confirmStyleModel: 'blue',
			confirmStyle: null,
			confirm: () => {},
			cancelText: '取消',
			cancelStyleModel: 'white',
			cancelStyle: null,
			cancel: () => {},
			hideTabBar: false
		}
	},
	methods: {
		open: function(dialogObj) {
			if (!dialogObj) {
				return
			}
			this.dataClear()
			const propertys = Reflect.ownKeys(dialogObj)
			propertys.forEach(x => {
				if (Reflect.has(this, x)) {
					Reflect.set(this, x, Reflect.get(dialogObj, x))
				}
			})
			if (this.hideTabBar) {
				uni.hideTabBar()
			}
			this.$refs.dragDrawerLandscape.open()
		},
		dataClear: function() {
			const dataPropertys = Reflect.ownKeys(this)
			dataPropertys.forEach(x => {
				switch (x) {
					case 'imgUrl':
						Reflect.set(this, x, '')
						break
					case 'title':
						Reflect.set(this, x, '')
						break
					case 'message':
						Reflect.set(this, x, '')
						break
					case 'mask':
						Reflect.set(this, x, true)
						break
					case 'showConfirmButton':
						Reflect.set(this, x, true)
						break
					case 'confirmText':
						Reflect.set(this, x, '确认')
						break
					case 'confirmStyleModel':
						Reflect.set(this, x, 'blue')
						break
					case 'confirmStyle':
						Reflect.set(this, x, null)
						break
					case 'confirm':
						Reflect.set(this, x, () => {})
						break
					case 'cancelText':
						Reflect.set(this, x, '取消')
						break
					case 'cancelStyleModel':
						Reflect.set(this, x, 'white')
						break
					case 'cancelStyle':
						Reflect.set(this, x, null)
						break
					case 'cancel':
						Reflect.set(this, x, () => {})
						break
					case 'hideTabBar':
						Reflect.set(this, x, false)
						break
					default:
						break
				}
			})
		},
		confirmClick: async function() {
			if (this.confirm) {
				await this.confirm()
			}
			this.close()
		},
		cancelClick: async function() {
			if (this.cancel) {
				await this.cancel()
			}
			this.close()
		},
		close: function() {
			this.$refs.dragDrawerLandscape && this.$refs.dragDrawerLandscape.close()
		},
		closed: function() {
			if (this.hideTabBar) {
				uni.showTabBar()
			}
		}
	}
}
</script>

<style lang="scss" scoped>
.drag-drawer-dialog-landscape {
	width: lpx2vw(375);
	height: 100%;
	display: flex;
	flex-flow: column nowrap;
	justify-content: space-between;
	align-items: center;
	.drag-drawer-dialog-landscape__content {
		width: 100%;
		height: 100%;
		display: flex;
		flex-flow: column nowrap;
		justify-content: flex-start;
		align-items: center;
		.drag-drawer-dialog-landscape__picture {
			margin-top: lpx2vh(30);
			margin-bottom: lpx2vh(20);
			width: lpx2vh(100);
			height: lpx2vh(100);
			background-repeat: no-repeat;
			background-size: cover;
			background-position: center;
		}
		.drag-drawer-dialog-landscape__title {
			width: 100%;
			display: flex;
			justify-content: center;
			font-family: PingFangSC-Medium;
			font-size: lpx2vh(16);
			color: #13171a;
		}
		.drag-drawer-dialog-landscape__message {
			width: lpx2vw(238);
			margin-top: lpx2vh(12);
			display: flex;
			justify-content: center;
			text-align: center;
			font-family: PingFangSC-Regular;
			font-size: lpx2vh(14);
			color: #80868c;
		}
	}
	.drag-drawer-dialog-landscape__foot {
		width: 100%;
		display: flex;
		flex-flow: column nowrap;
		justify-content: center;
		align-items: center;
		margin-bottom: lpx2vh(37);
		.drag-drawer-dialog-landscape__button {
			width: lpx2vw(319);
			height: lpx2vh(44);
			display: flex;
			justify-content: center;
			align-items: center;
			font-family: PingFangSC-Semibold;
			font-size: lpx2vh(16);
			border-radius: lpx2vw(8);
		}
		.drag-drawer-dialog-landscape__confirm {
			margin-bottom: lpx2vh(12);
		}
	}
}
</style>
