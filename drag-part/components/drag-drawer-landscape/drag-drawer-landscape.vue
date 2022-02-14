<template>
	<view v-if="visible" class="drag-drawer-landscape">
		<view class="drag-drawer-landscape__all-area">
			<view v-if="mask" class="drag-drawer-landscape__mask" @tap="close"></view>
			<view
				class="drag-drawer-landscape__area drag-drawer-landscape__transition"
				:class="{ 'drag-drawer-landscape__show': show }"
				:style="[drawerWidthStyle, drawerMove]"
			>
				<movable-area :style="movableAreaStyle" v-if="movable">
					<movable-view
						direction="horizontal"
						:x="x"
						:y="0"
						:style="drawerWidthStyle"
						@change="change"
					>
						<view :id="drawerId" class="drag-drawer-landscape__content">
							<view
								class="drag-drawer-landscape__line drag-drawer-landscape__line-show"
							></view>
							<slot></slot>
						</view>
					</movable-view>
				</movable-area>
				<view v-else :id="drawerId" class="drag-drawer-landscape__content">
					<view class="drag-drawer-landscape__line"></view>
					<slot></slot>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	name: 'DragDrawerLandscape',
	props: {
		mask: {
			type: Boolean,
			default: false
		},
		drawerId: {
			type: String,
			default: 'drag_drawer_landscape_movable_view_' + new Date().getTime()
		},
		movable: {
			type: Boolean,
			default: true
		}
	},
	mounted() {
		const screenWidthStr = uni.getSystemInfoSync().screenWidth
		const screenWidth = Number.isNaN(screenWidthStr)
			? 0
			: Number.parseFloat(screenWidthStr + '')
		this.maxMove = (80 * screenWidth) / 812
	},
	computed: {
		drawerWidthStyle() {
			return this.contentWidth ? { width: this.contentWidth + 'px' } : null
		},
		movableAreaStyle() {
			const width = this.contentWidth ? this.maxMove + this.contentWidth + 'px' : ''
			return width ? { width } : null
		}
	},
	data() {
		return {
			x: 0,
			old: {
				x: 0
			},
			contentWidth: 0,
			show: false,
			visible: false,
			maxMove: 0,
			timer: 0,
			drawerMove: null
		}
	},
	methods: {
		change: function({ detail: { x, source } }) {
			this.old.x = x
			if (x > 79) {
				this.close()
			} else {
				if (this.timer) {
					clearInterval(this.timer)
				}
				const that = this
				this.timer = setTimeout(() => {
					that.x = that.old.x
					that.$nextTick(() => {
						that.x = 0
					})
				}, 100)
			}
		},
		initWidth: function() {
			const that = this
			uni.createSelectorQuery()
				.select('#' + this.drawerId)
				.boundingClientRect(data => {
					that.contentWidth = Number.isNaN(data.width) ? 0 : Number.parseFloat(data.width)
					that.$nextTick(() => {
						that.show = true
					})
				})
				.exec()
		},
		open: function() {
			this.visible = true
			this.contentWidth = 0
			const that = this
			this.$nextTick(() => {
				that.initWidth()
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
.drag-drawer-landscape {
	position: fixed;
	right: 0;
	bottom: 0;
	z-index: 100;
	.drag-drawer-landscape__all-area {
		position: relative;
		.drag-drawer-landscape__mask {
			position: absolute;
			right: 0;
			bottom: 0;
			width: 100vw;
			height: 100vh;
			background-color: rgba(#000000, 0.6);
		}
		.drag-drawer-landscape__area {
			position: absolute;
			right: 0;
			bottom: 0;
			height: 100vh;
			overflow: hidden;
			transform: translate(100%, 0);
			z-index: 100;
			movable-area {
				width: auto;
				height: 100%;
				overflow: hidden;
				movable-view {
					width: auto;
					height: 100%;
					display: flex;
				}
			}
		}
		.drag-drawer-landscape__show {
			transform: translate(0, 0);
		}
		.drag-drawer-landscape__content {
			height: 100%;
			display: flex;
			align-items: center;
			background-color: #ffffff;
			border-radius: lpx2vw(8) 0 0 lpx2vw(8);
			.drag-drawer-landscape__line {
				width: lpx2vw(4);
				height: lpx2vh(40);
				margin-left: lpx2vw(6);
				margin-right: lpx2vw(8);
				border-radius: lpx2vw(2);
			}
			.drag-drawer-landscape__line-show {
				background-color: rgba(#000000, 0.1);
			}
		}
	}
}
.drag-drawer-landscape__transition {
	transition-property: transform;
	transition-duration: 0.3s;
	transition-timing-function: ease;
	transition-delay: 0s;
}
</style>
