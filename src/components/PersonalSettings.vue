<template>
	<div id="openproject_prefs" class="section">
		<SettingsTitle />
		<div id="openproject-content">
			<div id="toggle-openproject-navigation-link">
				<input id="openproject-link"
					type="checkbox"
					class="checkbox"
					:checked="state.navigation_enabled"
					@input="onNavigationChange">
				<label for="openproject-link">{{ t('integration_openproject', 'Enable navigation link') }}</label>
			</div>
			<br><br>
			<div v-if="connected" class="openproject-grid-form">
				<label class="openproject-connected">
					<CheckIcon class="openproject-connected-checkmark" :size="20" />
					{{ t('integration_openproject', 'Connected as {user}', { user: state.user_name }) }}
				</label><br>
				<Button id="openproject-rm-cred" @click="onLogoutClick">
					<template #icon>
						<CloseIcon :size="23" />
					</template>
					{{ t('integration_openproject', 'Disconnect from OpenProject') }}
				</Button>
			</div>
			<div v-if="connected" id="openproject-search-block">
				<input id="search-openproject"
					type="checkbox"
					class="checkbox"
					:checked="state.search_enabled"
					@input="onSearchChange">
				<label for="search-openproject">{{ t('integration_openproject', 'Enable unified search for tickets') }}</label>
				<br><br>
				<p v-if="state.search_enabled" class="settings-hint">
					<InformationVariant />
					{{ t('integration_openproject', 'Warning, everything you type in the search bar will be sent to your OpenProject instance.') }}
				</p>
				<input id="notification-openproject"
					type="checkbox"
					class="checkbox"
					:checked="state.notification_enabled"
					@input="onNotificationChange">
				<label for="notification-openproject">{{ t('integration_openproject', 'Enable notifications for activity in my work packages') }}</label>
			</div>
			<OAuthConnectButton v-else :is-admin-config-ok="state.admin_config_ok" />
		</div>
	</div>
</template>

<script>
import { loadState } from '@nextcloud/initial-state'
import { generateUrl } from '@nextcloud/router'
import axios from '@nextcloud/axios'
import CloseIcon from 'vue-material-design-icons/Close.vue'
import CheckIcon from 'vue-material-design-icons/Check.vue'
import InformationVariant from 'vue-material-design-icons/InformationVariant.vue'
import { showSuccess, showError } from '@nextcloud/dialogs'
import '@nextcloud/dialogs/styles/toast.scss'
import SettingsTitle from '../components/settings/SettingsTitle'
import OAuthConnectButton from './OAuthConnectButton'
import { translate as t } from '@nextcloud/l10n'
import { checkOauthConnectionResult } from '../utils'
import Button from '@nextcloud/vue/dist/Components/Button'

export default {
	name: 'PersonalSettings',

	components: {
		SettingsTitle, OAuthConnectButton, Button, CloseIcon, CheckIcon, InformationVariant,
	},

	data() {
		return {
			loading: false,
			state: loadState('integration_openproject', 'user-config'),
			oauthConnectionErrorMessage: loadState('integration_openproject', 'oauth-connection-error-message'),
			oauthConnectionResult: loadState('integration_openproject', 'oauth-connection-result'),
		}
	},

	computed: {
		connected() {
			if (!this.state.admin_config_ok) return false
			return this.state.token && this.state.token !== ''
				&& this.state.user_name && this.state.user_name !== ''
		},
	},

	mounted() {
		checkOauthConnectionResult(this.oauthConnectionResult, this.oauthConnectionErrorMessage)
	},

	methods: {
		onLogoutClick() {
			this.state.token = ''
			this.saveOptions({ token: this.state.token, token_type: '' })
		},
		onNotificationChange(e) {
			this.state.notification_enabled = e.target.checked
			this.saveOptions({ notification_enabled: this.state.notification_enabled ? '1' : '0' })
		},
		onSearchChange(e) {
			this.state.search_enabled = e.target.checked
			this.saveOptions({ search_enabled: this.state.search_enabled ? '1' : '0' })
		},
		onNavigationChange(e) {
			this.state.navigation_enabled = e.target.checked
			this.saveOptions({ navigation_enabled: this.state.navigation_enabled ? '1' : '0' })
		},
		saveOptions(values) {
			const req = {
				values,
			}
			const url = generateUrl('/apps/integration_openproject/config')
			axios.put(url, req)
				.then((response) => {
					showSuccess(t('integration_openproject', 'OpenProject options saved'))
					if (response.data.user_name !== undefined) {
						this.state.user_name = response.data.user_name
						if (this.state.token && response.data.user_name === '') {
							showError(t('integration_openproject', 'Incorrect access token'))
						}
					}
				})
				.catch((error) => {
					console.debug(error.response)
					const msg = error.response?.data?.errorMessage === 'Invalid token'
						? t('integration_openproject', 'Invalid token')
						: error.response?.data?.errorMessage === 'Not found'
							? t('integration_openproject', 'OpenProject instance not found')
							: error.response?.request?.responseText
					showError(
						t('integration_openproject', 'Failed to save OpenProject options')
						+ ': ' + msg
					)
				})
				.then(() => {
					this.loading = false
				})
		},
	},
}
</script>

<style scoped lang="scss">
#openproject-search-block {
	margin-top: 30px;
}

.openproject-grid-form label {
	line-height: 38px;
}

.openproject-grid-form input {
	width: 100%;
}

.openproject-grid-form {
	max-width: 600px;
	display: grid;
	grid-template: 1fr / 1fr 1fr;
	button .icon {
		margin-bottom: -1px;
	}
}

.openproject-connected {
	display: flex;
}

.openproject-connected-checkmark {
	padding-right: 8px;
	color: var(--color-success);
}

.settings-hint {
	display: flex;
}

#openproject_prefs .icon {
	display: inline-block;
	width: 32px;
}

#openproject_prefs .grid-form .icon {
	margin-bottom: -3px;
}

#openproject-content {
	margin-left: 40px;
}

#openproject-search-block .icon {
	width: 22px;
}

#openproject-content .oauth-connect--message {
	text-align: left !important;
}
</style>
