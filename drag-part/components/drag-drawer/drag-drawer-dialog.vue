<template>
	<drag-drawer
		:drawerId="drawerId"
		:mask="mask"
		ref="dragDrawer"
		:floatOnKeyboard="floatOnKeyboard"
		:movable="movable"
		@closed="closed"
	>
		<view class="drag-drawer-dialog">
			<view class="drag-drawer-dialog__content">
				<slot>
					<view
						v-if="imgUrl"
						class="drag-drawer-dialog__picture"
						:style="{ backgroundImage: 'url(' + imgUrl + ')' }"
					></view>
					<view class="drag-drawer-dialog__title">{{ title }}</view>
					<text v-if="message" class="drag-drawer-dialog__message">{{ message }}</text>
				</slot>
			</view>
			<view class="drag-drawer-dialog__foot">
				<view
					v-if="showConfirmButton"
					class="drag-drawer-dialog__button drag-drawer-dialog__confirm"
					:style="confirmButtonStyle"
					@tap="confirmClick"
				>
					{{ confirmText }}
				</view>
				<view
					class="drag-drawer-dialog__button"
					:style="cancelButtonStyle"
					@tap="cancelClick"
				>
					{{ cancelText }}
				</view>
			</view>
		</view>
	</drag-drawer>
</template>

<script>
const buleStyle = {
	backgroundColor: '#0084FF',
	color: '#ffffff'
}
const whiteStyle = {
	backgroundColor: '#ffffff',
	color: '#2B2F33',
	border: 100 / 375 + 'vw solid #CCD2D9'
}

import DragDrawer from './drag-drawer'
export default {
	name: 'DragDrawerDialog',
	components: {
		DragDrawer
	},
	props: {
		drawerId: {
			type: String,
			default: 'drag_drawer_dialog_' + new Date().getTime()
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
			hideTabBar: false,
			floatOnKeyboard: false,
			movable: true
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
			this.$refs.dragDrawer.open()
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
					case 'floatOnKeyboard':
						Reflect.set(this, x, false)
						break
					case 'movable':
						Reflect.set(this, x, true)
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
			this.$refs.dragDrawer && this.$refs.dragDrawer.close()
		},
		closed: function() {
			if (this.hideTabBar && !this.tabBarShow) {
				uni.showTabBar()
				this.$store.commit('setTabBarShow', true)
			}
		}
	}
}
</script>

<style lang="scss" scoped>
.drag-drawer-dialog {
	width: 100%;
	display: flex;
	flex-flow: column nowrap;
	justify-content: center;
	align-items: center;
	.drag-drawer-dialog__content {
		margin-top: px2vh(12);
		width: 100%;
		display: flex;
		flex-flow: column nowrap;
		justify-content: center;
		align-items: center;
		.drag-drawer-dialog__picture {
			margin-bottom: px2vh(20);
			width: px2vw(100);
			height: px2vw(100);
			background-repeat: no-repeat;
			background-size: cover;
			background-position: center;
		}
		.drag-drawer-dialog__title {
			width: px2vw(308);
			display: flex;
			justify-content: center;
			font-family: PingFangSC-Medium;
			font-size: px2vw(16);
			color: #13171a;
		}
		.drag-drawer-dialog__message {
			width: px2vw(308);
			margin-top: px2vh(12);
			display: flex;
			justify-content: center;
			text-align: center;
			font-family: PingFangSC-Regular;
			font-size: px2vw(14);
			color: #80868c;
		}
	}
	.drag-drawer-dialog__foot {
		margin-top: px2vh(20);
		margin-bottom: px2vh(25);
		margin-bottom: calc(calc(25 / 812 * 100vh) + constant(safe-area-inset-bottom));
		margin-bottom: calc(calc(25 / 812 * 100vh) + env(safe-area-inset-bottom));
		width: 100%;
		display: flex;
		flex-flow: column nowrap;
		justify-content: center;
		align-items: center;
		.drag-drawer-dialog__button {
			width: px2vw(319);
			height: px2vh(44);
			display: flex;
			justify-content: center;
			align-items: center;
			font-family: PingFangSC-Semibold;
			font-size: px2vw(16);
			border-radius: px2vw(8);
		}
		.drag-drawer-dialog__confirm {
			margin-bottom: px2vh(12);
		}
	}
}
</style>
