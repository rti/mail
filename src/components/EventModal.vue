<template>
	<Modal @close="onClose">
		<div class="modal-content">
			<h2>{{ t('mail', 'Create event') }}</h2>
			<Multiselect
				v-model="selectedCalendar"
				label="displayname"
				track-by="url"
				:allow-empty="false"
				:options="calendars" />
			<input v-model="eventTitle" type="text">
			<DatetimePicker type="datetime" :show-timezone-select="true" :timezone-id="startTimezoneId" />
			<DatetimePicker type="datetime" :show-timezone-select="true" :timezone-id="endTimezoneId" />
			<button class="primary" @click="onSave">
				{{ t('mail', 'Create') }}
			</button>
		</div>
	</Modal>
</template>

<script>
import DatetimePicker from '@nextcloud/vue/dist/Components/DatetimePicker'
import jstz from 'jstz'
import Modal from '@nextcloud/vue/dist/Components/Modal'
import Multiselect from '@nextcloud/vue/dist/Components/Multiselect'

import { getUserCalendars } from '../service/DAVService'
import logger from '../logger'

export default {
	name: 'EventModal',
	components: {
		DatetimePicker,
		Modal,
		Multiselect,
	},
	props: {
		envelope: {
			type: Object,
			required: true,
		},
	},
	data() {
		// Try to determine the current timezone, and fall back to UTC otherwise
		const defaultTimezone = jstz.determine()
		const defaultTimezoneId = defaultTimezone ? defaultTimezone.name() : 'UTC'

		return {
			calendars: [],
			eventTitle: this.envelope.subject,
			startTimezoneId: defaultTimezoneId,
			endTimezoneId: defaultTimezoneId,
			saving: false,
			selectedCalendar: undefined,
		}
	},
	created() {
		logger.debug('creating event from envelope', {
			envelope: this.envelope,
		})
	},
	async mounted() {
		this.calendars = (await getUserCalendars()).filter(c => c.writable)

		if (this.calendars.length) {
			this.selectedCalendar = this.calendars[0]
		}
	},
	methods: {
		onClose() {
			this.$emit('close')
		},
		async onSave() {
			this.saving = true

			try {
				console.info('create event', {
					calendar: this.selectedCalendar,
					eventTitle: this.eventTitle,
					startDate: this.startDate,
					startTimezone: this.startTimezoneId,
					endTimezone: this.endTimezoneId,
				})
			} finally {
				this.saving = false
			}
		},
	},
}
</script>

<style lang="scss">
.modal-container {
	width: calc(100vw - 120px) !important;
	height: calc(100vh - 120px) !important;
	max-width: 600px !important;
	max-height: 500px !important;
}
</style>
