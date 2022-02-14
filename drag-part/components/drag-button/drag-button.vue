<template>
	<view
		class="drag-button"
		:class="{ 'drag-button__spread': !disable && !locked }"
		:style="buttonStyle"
	>
		<view class="drag-button__area">
			<view
				class="drag-button__button"
				:class="{
					'drag-button__button-spread': spread && !locked,
					'drag-button__button-locked': locked
				}"
				:id="buttonId"
				:buttonConfig="buttonConfig"
				:change:buttonConfig="hammerjs.updateConfig"
			>
				<view class="drag-button__button-icon-area" @tap="buttonIconTap">
					<view
						class="drag-button__button-icon"
						:style="{ backgroundImage: 'url(' + buttonIcon + ')' }"
					></view>
				</view>
				<view class="drag-button__text" @tap="buttonTap">
					<view class="drag-button__title">{{ buttonTitle }}</view>
					<view class="drag-button__desc">{{ buttonDesc }}</view>
				</view>
			</view>
		</view>
	</view>
</template>

<script>
export default {
	name: 'DragButton',
	props: {
		buttonId: {
			type: String,
			default: 'drag-bottun-' + new Date().getTime()
		},
		buttonTitle: {
			type: String,
			default: ''
		},
		buttonDesc: {
			type: String,
			default: ''
		},
		buttonIcon: {
			type: String,
			default: ''
		},
		topMax: {
			type: [String, Number],
			default: 0
		},
		bottomMax: {
			type: [String, Number],
			default: 812
		},
		spread: {
			type: Boolean,
			default: true
		},
		locked: {
			type: Boolean,
			default: false
		}
	},
	watch: {
		spread: {
			handler: function(newVal, oldVal) {
				const that = this
				if (newVal) {
					this.disable = false
				} else {
					setTimeout(() => {
						that.disable = true
					}, 300)
				}
			},
			immediate: true
		}
	},
	computed: {
		buttonConfig() {
			const buttonId = this.buttonId
			const screenHeightStr = uni.getSystemInfoSync().screenHeight
			const screenHeight = Number.isNaN(screenHeightStr)
				? 0
				: Number.parseFloat(screenHeightStr + '')
			const topMax = Number.isNaN(this.topMax)
				? 0
				: (Number.parseFloat(this.topMax + '') * screenHeight) / 812
			const bottomMax = Number.isNaN(this.bottomMax)
				? 0
				: (Number.parseFloat(this.bottomMax + '') * screenHeight) / 812
			return {
				buttonId,
				topMax,
				bottomMax
			}
		}
	},
	data() {
		return {
			direction: 'horizontal',
			x: 0,
			y: 0,
			buttonStyle: null,
			disable: false
		}
	},
	methods: {
		buttonTap: function() {
			if (this.disable || this.locked) {
				return
			}
			this.$emit('buttonTap')
		},
		updateButtonStyle: function(style) {
			this.buttonStyle = style
		},
		buttonIconTap: function() {
			this.$emit('buttonIconTap')
		}
	}
}
</script>

<script lang="renderjs" module="hammerjs">
import Hammer from 'hammerjs'
export default{
	mounted() {
		this.initHammer(this.buttonConfig)
	},
	data() {
		return {
			moveable: false,
			currentTop: 0,
			currentBottom: 0,
			distance: 0
		}
	},
	methods: {
		initHammer: function(config) {
			const that = this
			const dom = document.getElementById(config.buttonId)
			const hammer = new Hammer(dom)
			hammer.get('pan').set({
				direction: Hammer.DIRECTION_VERTICAL
			})
			hammer.on('panup pandown panstart pancancel panend', function(ev) {
				const func = that[ev.type]
				func && func(ev)
			})
		},
		updateConfig: function(newValue, oldValue, ownerInstance, instance) {
			if (newValue.buttonId && newValue.buttonId != oldValue.buttonId) {
				this.initHammer(newValue)
			}
		},
		panstart: function(ev) {
			const dom = document.getElementById(this.buttonConfig.buttonId)
			const domArea = dom.parentNode.parentNode
			this.currentTop = domArea.offsetTop + this.distance
			this.currentBottom = domArea.offsetTop + this.distance + dom.scrollHeight
			this.$ownerInstance.callMethod('updateButtonStyle', { top: this.currentTop + 'px' })
			this.distance = 0
		},
		panup: function(ev) {
			this.buttonMove(ev.deltaY)
		},
		pandown: function(ev) {
			this.buttonMove(ev.deltaY)
		},
		pancancel: function(ev) {

		},
		panend: function(ev) {

		},
		buttonMove: function(distance) {
			if (
				this.currentTop + distance > this.buttonConfig.topMax &&
				this.currentBottom + distance < this.buttonConfig.bottomMax
			) {
				this.$ownerInstance.callMethod('updateButtonStyle', {
					top: this.currentTop + 'px',
					transform: 'translateY(' + distance + 'px)'
				})
				this.distance = distance
			}
		}
	}
}
</script>

<style lang="scss" scoped>
.drag-button {
	position: absolute;
	width: px2vw(50);
	height: px2vw(60);
	overflow: hidden;
	.drag-button__area {
		width: 100%;
		height: 100%;
		position: relative;
		.drag-button__button {
			position: absolute;
			top: 50%;
			right: 0;
			transform: translate(calc(112 / 375 * 100vw), -50%);
			width: px2vw(162);
			height: px2vw(50);
			border-radius: px2vw(100) 0 0 px2vw(100);
			background-color: #ffcf2d;
			display: flex;
			box-shadow: 0 px2vw(2) px2vw(4) 0 rgba(#bc940d, 0.19);
			transition-property: transform;
			transition-timing-function: ease;
			transition-delay: 0s;
			transition-duration: 0.3s;
			.drag-button__button-icon-area {
				width: px2vw(50);
				height: px2vw(50);
				display: flex;
				justify-content: center;
				align-items: center;
				.drag-button__button-icon {
					width: px2vw(24);
					height: px2vw(24);
					background-repeat: no-repeat;
					background-size: cover;
					background-position: center;
				}
			}
			.drag-button__text {
				width: px2vw(112);
				height: px2vw(50);
				display: flex;
				flex-flow: column nowrap;
				justify-content: space-between;
				align-items: flex-start;
				.drag-button__title {
					display: flex;
					line-height: px2vw(16);
					font-family: PingFangSC-Semibold;
					font-size: px2vw(15);
					color: #040200;
					margin-top: px2vw(8);
				}
				.drag-button__desc {
					display: flex;
					line-height: px2vw(12);
					font-family: PingFangSC-Regular;
					font-size: px2vw(12);
					color: #040200;
					margin-bottom: px2vw(9);
					opacity: 0.5;
				}
			}
		}
		.drag-button__button-spread {
			transform: translate(0, -50%);
		}
		.drag-button__button-locked {
			background-color: #dadada;
			box-shadow: 0 px2vw(2) px2vw(4) 0 rgba(#6c6c6c, 0.19);
		}
	}
}
.drag-button__spread {
	width: px2vw(180);
	height: px2vw(60);
}
</style>
