<template>
	<view v-if="visible" class="drag-drawer">
		<view class="drag-drawer__all-area">
			<view v-if="mask" class="drag-drawer__mask" @tap="close"></view>
			<view
				class="drag-drawer__area drag-drawer__transition"
				:class="{ 'drag-drawer__show': show }"
				:style="[drawerHeightStyle, drawerMove]"
			>
				<movable-area :style="movableAreaStyle" v-if="movable">
					<movable-view
						direction="vertical"
						:x="0"
						:y="y"
						:style="drawerHeightStyle"
						@change="change"
					>
						<view :id="drawerId" class="drag-drawer__content">
							<view class="drag-drawer__line drag-drawer__line-show"></view>
							<slot></slot>
						</view>
					</movable-view>
				</movable-area>
				<view v-else :id="drawerId" class="drag-drawer__content">
					<view class="drag-drawer__line"></view>
					<slot></slot>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	name: 'DragDrawer',
	props: {
		mask: {
			type: Boolean,
			default: false
		},
		drawerId: {
			type: String,
			default: 'drag_drawer_movable_view_' + new Date().getTime()
		},
		movable: {
			type: Boolean,
			default: true
		},
		floatOnKeyboard: {
			type: Boolean,
			default: false
		}
	},
	mounted() {
		const screenHeightStr = uni.getSystemInfoSync().screenHeight
		const screenHeight = Number.isNaN(screenHeightStr)
			? 0
			: Number.parseFloat(screenHeightStr + '')
		this.maxMove = (80 * screenHeight) / 812
	},
	watch: {
		floatOnKeyboard: {
			handler: function(newVal, oldVal) {
				if (!newVal) {
					return
				}
				const that = this
				uni.offKeyboardHeightChange(function() {})
				uni.onKeyboardHeightChange(res => {
					const keyboardHeight = Number.isNaN(res.height)
						? 0
						: Number.parseFloat(res.height)
					that.$store.commit('setKeyboardHeight', keyboardHeight)
					that.drawerMove = !keyboardHeight
						? null
						: {
								transform: 'translate(0, -' + keyboardHeight + 'px)',
								transitionDuration: '0.35s',
								transitionTimingFunction: 'ease-out',
								transitionDelay: '0.1s'
						  }
				})
			},
			immediate: true
		}
	},
	computed: {
		drawerHeightStyle() {
			return this.contentHeight ? { height: this.contentHeight + 'px' } : null
		},
		movableAreaStyle() {
			const height = this.contentHeight ? this.maxMove + this.contentHeight + 'px' : ''
			return height ? { height } : null
		}
	},
	data() {
		return {
			y: 0,
			old: {
				y: 0
			},
			contentHeight: 0,
			show: false,
			visible: false,
			maxMove: 0,
			timer: 0,
			drawerMove: null
		}
	},
	methods: {
		change: function({ detail: { y, source } }) {
			this.old.y = y
			if (y > 79) {
				this.close()
			} else {
				if (this.timer) {
					clearInterval(this.timer)
				}
				const that = this
				this.timer = setTimeout(() => {
					that.y = that.old.y
					that.$nextTick(() => {
						that.y = 0
					})
				}, 100)
			}
		},
		initHeight: function() {
			const that = this
			uni.createSelectorQuery()
				.select('#' + that.drawerId)
				.boundingClientRect(data => {
					that.contentHeight = Number.isNaN(data.height)
						? 0
						: Number.parseFloat(data.height)
					that.$nextTick(() => {
						that.show = true
					})
				})
				.exec()
		},
		open: function() {
			this.visible = true
			this.contentHeight = 0
			const that = this
			this.$nextTick(() => {
				that.initHeight()
			})
		},
		close: function() {
			this.show = false
			const that = this
			setTimeout(() => {
				that.$emit('closed')
				that.visible = false
			}, 300)
		}
	}
}
</script>

<style lang="scss" scoped>
.drag-drawer {
	position: fixed;
	left: 0;
	bottom: 0;
	z-index: 100;
	.drag-drawer__all-area {
		position: relative;
		.drag-drawer__mask {
			position: absolute;
			left: 0;
			bottom: 0;
			bottom: constant(safe-area-inset-bottom);
			bottom: env(safe-area-inset-bottom);
			width: 100vw;
			height: 100vh;
			height: calc(100vh - constant(safe-area-inset-bottom));
			height: calc(100vh - env(safe-area-inset-bottom));
			background-color: rgba(#000000, 0.6);
		}
		.drag-drawer__area {
			position: absolute;
			left: 0;
			bottom: 0;
			width: 100vw;
			overflow: hidden;
			transform: translate(0, 100%);
			z-index: 100;
			movable-area {
				width: 100%;
				height: auto;
				overflow: hidden;
				movable-view {
					width: 100%;
					height: auto;
					display: flex;
				}
			}
		}
		.drag-drawer__show {
			transform: translate(0, 0);
		}
		.drag-drawer__content {
			width: 100%;
			display: flex;
			flex-flow: column nowrap;
			align-items: center;
			background-color: #ffffff;
			border-radius: px2vw(8) px2vw(8) 0 0;
			.drag-drawer__line {
				width: px2vw(40);
				height: px2vh(4);
				margin-top: px2vh(6);
				margin-bottom: px2vh(8);
				border-radius: px2vh(2);
			}
			.drag-drawer__line-show {
				background-color: rgba(#000000, 0.1);
			}
		}
	}
}
.drag-drawer__transition {
	transition-property: transform;
	transition-duration: 0.3s;
	transition-timing-function: ease;
	transition-delay: 0s;
}
</style>
