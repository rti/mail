<template>
	<Modal @close="onClose">
		<div class="modal-content">
			<h2>{{ t('mail', 'Create event') }}</h2>
			<input v-model="eventTitle" type="text">
			<button class="primary">
				{{ t('mail', 'Create') }}
			</button>
		</div>
	</Modal>
</template>

<script>
import Modal from '@nextcloud/vue/dist/Components/Modal'

import logger from '../logger'

export default {
	name: 'EventModal',
	components: {
		Modal,
	},
	props: {
		envelope: {
			type: Object,
			required: true,
		},
	},
	data() {
		return {
			eventTitle: this.envelope.subject,
		}
	},
	created() {
		logger.debug('creating event from envelope', {
			envelope: this.envelope,
		})
	},
	methods: {
		onClose() {
			this.$emit('close')
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
