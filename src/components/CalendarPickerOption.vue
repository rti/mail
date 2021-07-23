<template>
	<div class="calendar-picker-option">
		<div
			class="calendar-picker-option__color-indicator"
			:style="{ backgroundColor: color }" />

		<span
			class="calendar-picker-option__label">
			{{ displayname }}
		</span>
	</div>
</template>

<script>
export default {
	name: 'CalendarPickerOption',
	props: {
		color: {
			type: String,
			required: true,
		},
		displayname: {
			type: String,
			required: true,
		},
	},
}
</script>

<style lang="scss" scoped>
.calendar-picker-option {
	width: 100%;
	display: flex;
	align-items: center;

	&__color-indicator {
		width: 12px;
		height: 12px;
		border-radius: 50%;
		border: none;
		margin-right: 8px;
		flex-basis: 12px;
		flex-shrink: 0;
	}

	&__label {
		overflow: hidden;
		text-overflow: ellipsis;
		flex-grow: 1;
	}
}

</style>
