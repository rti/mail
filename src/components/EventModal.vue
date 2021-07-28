<template>
	<Modal @close="onClose">
		<div class="modal-content">
			<h2>{{ t('mail', 'Create event') }}</h2>
			<div class="eventTitle">
				<input v-model="eventTitle" type="text">
			</div>
			<div class="dateTimePicker">
				<DatetimePicker type="datetime" :show-timezone-select="true" :timezone-id="startTimezoneId" />
				<DatetimePicker type="datetime" :show-timezone-select="true" :timezone-id="endTimezoneId" />
			</div>
			<div v-if="!isReadOnly" class="all-day">
				<input
					id="allDay"
					:checked="isAllDay"
					type="checkbox"
					class="checkbox"
					:disabled="!canModifyAllDay"
					@change="toggleAllDay">
				<label
					for="allDay">
					{{ t('mail', 'All day') }}
				</label>
			</div>
			<Multiselect
				v-model="selectedCalendar"
				label="displayname"
				track-by="url"
				:allow-empty="false"
				:options="calendars">
				<template #option="{option}">
					<CalendarPickerOption
						v-bind="option" />
				</template>
				<template #singleLabel="{option}">
					<CalendarPickerOption
						:display-icon="true"
						v-bind="option" />
				</template>
			</Multiselect>
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
import CalendarPickerOption from './CalendarPickerOption'

export default {
	name: 'EventModal',
	components: {
		CalendarPickerOption,
		DatetimePicker,
		Modal,
		Multiselect,
	},
	props: {
		envelope: {
			type: Object,
			required: true,
		},
		isAllDay: {
			type: Boolean,
			required: true,
		},
		canModifyAllDay: {
			type: Boolean,
			required: true,
		},
		isReadOnly: {
			type: Boolean,
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
		toggleAllDay() {
			if (!this.canModifyAllDay) {
				return
			}
			this.$emit('toggleAllDay')
		},
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
	max-width: 490px !important;
	max-height: 500px !important;
}
.modal-content {
	padding: 30px 30px 20px !important;
}
input {
	width: 100%;
}
::v-deep input[type='text'].multiselect__input {
	padding: 0 !important;
}
::v-deep .multiselect__single {
	margin-left: -18px;
	width: 100px;
}
::v-deep .multiselect__tags {
	border: none !important;
}
.all-day {
	margin-left: -1px;
	margin-top: 5px;
	margin-bottom: 5px;
}
.eventTitle {
	margin-bottom: 5px;
}
.primary {
	height: 44px !important;
	float: right;
}
::v-deep .mx-datepicker {
	width: 213px;
}
</style>
