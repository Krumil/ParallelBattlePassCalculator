<template>
	<label :for="forId">
		{{ labelText }}
		<span class="info-icon" @mouseover="showTooltip" @mouseout="hideTooltip"
			>ℹ️
			<div class="tooltip" v-if="showingTooltip">{{ tooltipText }}</div>
		</span>
	</label>
</template>

<script>
	export default {
		props: {
			forId: String,
			labelText: String,
			tooltipText: String
		},
		data() {
			return {
				showingTooltip: false
			};
		},
		methods: {
			showTooltip() {
				this.showingTooltip = true;
			},
			hideTooltip() {
				this.showingTooltip = false;
			}
		}
	};
</script>

<style scoped>
	.info-icon {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 16px;
		height: 16px;
		position: relative;
		cursor: pointer;
		margin-left: 5px;
		border-radius: 50%;
		border: 1px solid #def141;
		font-size: 10px;

		.tooltip {
			border-radius: 50%;
			border: 1px solid #def141;
			position: absolute;
			background-color: #333;
			color: white;
			padding: 5px 10px;
			border-radius: 5px;
			top: -40px;
			left: 50%;
			transform: translateX(-50%);
			white-space: nowrap;
			z-index: 10;
			opacity: 0.9;
			font-size: 14px;
		}
	}
</style>
