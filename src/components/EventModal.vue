<template>
	<Modal @close="onClose">
		<div class="modal-content">
			<h2>{{ t('mail', 'Create event') }}</h2>
			<div class="eventTitle">
				<input v-model="eventTitle" type="text">
			</div>
			<Multiselect
				v-model="selectedCalendar"
				label="displayname"
				track-by="url"
				:allow-empty="false"
				:options="calendars" />
			<br>
			<DatetimePicker type="datetime" :show-timezone-select="true" :timezone-id="startTimezoneId" />
			<DatetimePicker type="datetime" :show-timezone-select="true" :timezone-id="endTimezoneId" />
			<br>
			<button class="primary" @click="onSave">
				{{ t('mail', 'Create') }}
			</button>
		</div>
	</Modal>
</template>

<script>
import { createEvent, getTimezoneManager } from 'calendar-js'
import DatetimePicker from '@nextcloud/vue/dist/Components/DatetimePicker'
import DateTimeValue from 'calendar-js/src/values/dateTimeValue'
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

				// TODO: the tz manager is not initialized, it won't find any timezones
				//       https://github.com/nextcloud/calendar-js/issues/273
				const timezoneManager = getTimezoneManager()
				const startTimezone = timezoneManager.getTimezoneForId(this.startTimezoneId)
				const startDateTime = DateTimeValue
					.fromJSDate(this.startDate, true)
					.getInTimezone(startTimezone)
				const endTimezone = timezoneManager.getTimezoneForId(this.endTimezoneId)
				const endDateTime = DateTimeValue
					.fromJSDate(this.endDate, true)
					.getInTimezone(endTimezone)

				const calendar = createEvent(startDateTime, endDateTime)
				for (const vObject of calendar.getVObjectIterator()) {
					vObject.undirtify()
				}

				console.info('object/valendar created', { calendar })
			} finally {
				this.saving = false
			}
		},
	},
}
</script>

<style lang="scss" scoped>
::v-deep .modal-wrapper .modal-container {
	width: calc(100vw - 120px) !important;
	height: calc(100vh - 120px) !important;
	max-width: 600px !important;
	max-height: 500px !important;
}
.modal-content {
	padding: 30px 40px 20px !important;
}
input {
	width: 100% !important;
}
::v-deep .multiselect.multiselect--single {
	width: 100% !important;
}
</style>
