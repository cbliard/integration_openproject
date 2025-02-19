<template>
	<div id="openproject_prefs" class="section">
		<SettingsTitle />
		<div class="openproject-server-host">
			<FormHeading index="1"
				:title="t('integration_openproject', 'OpenProject server')"
				:is-complete="isServerHostFormComplete" />
			<FieldValue v-if="isServerHostFormInView"
				is-required
				class="pb-1"
				:title="t('integration_openproject', 'OpenProject host')"
				:value="state.oauth_instance_url" />
			<TextInput v-else
				id="openproject-oauth-instance"
				ref="openproject-oauth-instance-input"
				v-model="serverHostUrlForEdit"
				is-required
				:read-only="isServerHostUrlReadOnly"
				class="pb-1"
				:label="t('integration_openproject', 'OpenProject host')"
				place-holder="https://www.my-openproject.com"
				:hint-text="t('integration_openproject', 'Please introduce your OpenProject host name')"
				:error-message="serverHostErrorMessage"
				@click="isServerHostUrlReadOnly = false"
				@input="isOpenProjectInstanceValid = null" />
			<div class="form-actions">
				<Button v-if="isServerHostFormInView"
					data-test-id="reset-server-host-btn"
					@click="setServerHostFormToEditMode">
					<template #icon>
						<PencilIcon :size="20" />
					</template>
					{{ t('integration_openproject', 'Edit server information') }}
				</Button>
				<Button v-if="isServerHostFormComplete && isServerHostFormInEdit"
					class="mr-2"
					data-test-id="cancel-edit-server-host-btn"
					@click="setServerHostFormToViewMode">
					{{ t('integration_openproject', 'Cancel') }}
				</Button>
				<Button v-if="isServerHostFormInEdit"
					type="primary"
					data-test-id="submit-server-host-form-btn"
					:disabled="!serverHostUrlForEdit || serverHostUrlForEdit === state.oauth_instance_url"
					@click="saveOpenProjectHostUrl">
					<template #icon>
						<LoadingIcon v-if="loadingServerHostForm" class="loading-spinner" :size="20" />
						<CheckBoldIcon v-else :size="20" />
					</template>
					{{ t('integration_openproject', 'Save') }}
				</Button>
			</div>
		</div>
		<div class="openproject-oauth-values">
			<FormHeading index="2"
				:title="t('integration_openproject', 'OpenProject OAuth settings')"
				:is-complete="isOPOAuthFormComplete"
				:is-disabled="isOPOAuthFormInDisableMode" />
			<div v-if="isServerHostFormComplete">
				<FieldValue v-if="isOPOAuthFormInView"
					is-required
					:value="state.client_id"
					:title="t('integration_openproject', 'OpenProject OAuth client ID')" />
				<TextInput v-else
					id="openproject-oauth-client-id"
					v-model="state.client_id"
					class="py-1"
					is-required
					:label="t('integration_openproject', 'OpenProject OAuth client ID')"
					:hint-text="openProjectClientHint" />
				<FieldValue v-if="isOPOAuthFormInView"
					is-required
					class="pb-1"
					encrypt-value
					:title="t('integration_openproject', 'OpenProject OAuth client secret')"
					:value="state.client_secret" />
				<TextInput v-else
					id="openproject-oauth-client-secret"
					v-model="state.client_secret"
					is-required
					class="py-1"
					:label="t('integration_openproject', 'OpenProject OAuth client secret')"
					:hint-text="openProjectClientHint" />
				<div class="form-actions">
					<Button v-if="isOPOAuthFormComplete && isOPOAuthFormInView"
						data-test-id="reset-op-oauth-btn"
						@click="resetOPOAuthClientValues">
						<template #icon>
							<AutoRenewIcon :size="20" />
						</template>
						{{ t('integration_openproject', 'Replace OpenProject OAuth values') }}
					</Button>
					<Button v-else
						data-test-id="submit-op-oauth-btn"
						type="primary"
						:disabled="!state.client_id || !state.client_secret"
						@click="saveOPOAuthClientValues">
						<template #icon>
							<LoadingIcon v-if="loadingOPOauthForm" class="loading-spinner" :size="20" />
							<CheckBoldIcon v-else :size="20" />
						</template>
						{{ t('integration_openproject', 'Save') }}
					</Button>
				</div>
			</div>
		</div>
		<div class="nextcloud-oauth-values">
			<FormHeading index="3"
				:title="t('integration_openproject', 'Nextcloud OAuth client')"
				:is-complete="isNcOAuthFormComplete"
				:is-disabled="isNcOAuthFormInDisableMode" />
			<div v-if="state.nc_oauth_client">
				<TextInput v-if="isNcOAuthFormInEdit"
					id="nextcloud-oauth-client-id"
					v-model="state.nc_oauth_client.clientId"
					class="py-1"
					read-only
					is-required
					with-copy-btn
					:label="t('integration_openproject', 'Nextcloud OAuth client ID')"
					:hint-text="nextcloudClientHint" />
				<FieldValue v-else
					:title="t('integration_openproject', 'Nextcloud OAuth client ID')"
					:value="state.nc_oauth_client.clientId"
					is-required />
				<TextInput v-if="isNcOAuthFormInEdit"
					id="nextcloud-oauth-client-secret"
					v-model="state.nc_oauth_client.clientSecret"
					class="py-1"
					read-only
					is-required
					with-copy-btn
					:label="t('integration_openproject', 'Nextcloud OAuth client secret')"
					:hint-text="nextcloudClientHint" />
				<FieldValue v-else
					:title="t('integration_openproject', 'Nextcloud OAuth client secret')"
					is-required
					encrypt-value
					with-inspection
					:value="ncClientSecret" />
				<div class="form-actions">
					<Button v-if="isNcOAuthFormInEdit"
						type="primary"
						:disabled="!ncClientId || !ncClientSecret"
						data-test-id="submit-nc-oauth-values-form-btn"
						@click="setNCOAuthFormToViewMode">
						<template #icon>
							<CheckBoldIcon :size="20" />
						</template>
						{{ t('integration_openproject', 'Yes, I have copied these values') }}
					</Button>
					<Button v-else
						data-test-id="reset-nc-oauth-btn"
						@click="resetNcOauthValues">
						<template #icon>
							<AutoRenewIcon :size="20" />
						</template>
						{{ t('integration_openproject', 'Replace Nextcloud OAuth values') }}
					</Button>
				</div>
			</div>
		</div>
		<Button id="reset-all-app-settings-btn"
			type="error"
			@click="resetAllAppValuesConfirmation">
			<template #icon>
				<RestoreIcon :size="20" />
			</template>
			{{ t('integration_openproject', 'Reset') }}
		</Button>
	</div>
</template>

<script>
import util from 'util'
import axios from '@nextcloud/axios'
import '@nextcloud/dialogs/styles/toast.scss'
import { loadState } from '@nextcloud/initial-state'
import { generateUrl } from '@nextcloud/router'
import { showSuccess, showError } from '@nextcloud/dialogs'
import CheckBoldIcon from 'vue-material-design-icons/CheckBold.vue'
import PencilIcon from 'vue-material-design-icons/Pencil.vue'
import LoadingIcon from 'vue-material-design-icons/Loading.vue'
import RestoreIcon from 'vue-material-design-icons/Restore.vue'
import AutoRenewIcon from 'vue-material-design-icons/Autorenew.vue'
import TextInput from './admin/TextInput'
import FieldValue from './admin/FieldValue'
import FormHeading from './admin/FormHeading'
import SettingsTitle from '../components/settings/SettingsTitle'
import { F_MODES } from './../utils'
import Button from '@nextcloud/vue/dist/Components/Button'

export default {
	name: 'AdminSettings',

	components: {
		Button,
		FieldValue,
		FormHeading,
		TextInput,
		SettingsTitle,
		CheckBoldIcon,
		PencilIcon,
		LoadingIcon,
		AutoRenewIcon,
		RestoreIcon,
	},
	data() {
		return {
			formMode: {
				// server host form is never disabled.
				// it's either editable or view only
				server: F_MODES.EDIT,
				opOauth: F_MODES.DISABLE,
				ncOauth: F_MODES.DISABLE,
			},
			isFormCompleted: {
				server: false, opOauth: false, ncOauth: false,
			},
			loadingServerHostForm: false,
			loadingOPOauthForm: false,
			isOpenProjectInstanceValid: null,
			state: loadState('integration_openproject', 'admin-config'),
			isAdminConfigOk: loadState('integration_openproject', 'admin-config-status'),
			serverHostUrlForEdit: null,
			isServerHostUrlReadOnly: true,
		}
	},
	computed: {
		ncClientId() {
			return this.state.nc_oauth_client?.clientId
		},
		ncClientSecret() {
			return this.state.nc_oauth_client?.clientSecret
		},
		serverHostErrorMessage() {
			if (
				this.serverHostUrlForEdit === ''
				|| this.isOpenProjectInstanceValid === null
				|| this.isOpenProjectInstanceValid
			) return null
			return t('integration_openproject', 'Please introduce a valid OpenProject host name')
		},
		isServerHostFormComplete() {
			return this.isFormCompleted.server
		},
		isOPOAuthFormComplete() {
			return this.isFormCompleted.opOauth
		},
		isNcOAuthFormComplete() {
			return this.isFormCompleted.ncOauth
		},
		isServerHostFormInView() {
			return this.formMode.server === F_MODES.VIEW
		},
		isOPOAuthFormInView() {
			return this.formMode.opOauth === F_MODES.VIEW
		},
		isServerHostFormInEdit() {
			return this.formMode.server === F_MODES.EDIT
		},
		isNcOAuthFormInEdit() {
			return this.formMode.ncOauth === F_MODES.EDIT
		},
		isOPOAuthFormInDisableMode() {
			return this.formMode.opOauth === F_MODES.DISABLE
		},
		isNcOAuthFormInDisableMode() {
			return this.formMode.ncOauth === F_MODES.DISABLE
		},
		adminFileStorageHref() {
			let hostPart = ''
			const urlPart = '%sadmin/settings/storages'
			if (this.state?.oauth_instance_url.endsWith('/')) {
				hostPart = this.state.oauth_instance_url
			} else hostPart = this.state.oauth_instance_url + '/'
			return util.format(urlPart, hostPart)
		},
		openProjectClientHint() {
			const linkText = t('integration_openproject', 'Administration > File storages')
			const htmlLink = `<a class="link" href="${this.adminFileStorageHref}" target="_blank" title="${linkText}">${linkText}</a>`
			return t('integration_openproject', 'Go to your OpenProject {htmlLink} as an Administrator and start the setup and copy the values here.', { htmlLink }, null, { escape: false, sanitize: false })
		},
		nextcloudClientHint() {
			const linkText = t('integration_openproject', 'Administration > File storages')
			const htmlLink = `<a class="link" href="${this.adminFileStorageHref}" target="_blank" title="${linkText}">${linkText}</a>`
			return t('integration_openproject', 'Copy the following values back into the OpenProject {htmlLink} as an Administrator.', { htmlLink }, null, { escape: false, sanitize: false })
		},
	},
	created() {
		if (this.state) {
			if (this.state.oauth_instance_url) {
				this.formMode.server = F_MODES.VIEW
				this.isFormCompleted.server = true
			}
			if (!!this.state.client_id && !!this.state.client_secret) {
				this.formMode.opOauth = F_MODES.VIEW
				this.isFormCompleted.opOauth = true
			}
			if (this.state.oauth_instance_url) {
				if (!this.state.client_id || !this.state.client_secret) {
					this.formMode.opOauth = F_MODES.EDIT
				}
			}
			if (this.state.nc_oauth_client) {
				this.formMode.ncOauth = F_MODES.VIEW
				this.isFormCompleted.ncOauth = true
			}
		}
	},
	methods: {
		setServerHostFormToViewMode() {
			this.formMode.server = F_MODES.VIEW
		},
		setServerHostFormToEditMode() {
			this.formMode.server = F_MODES.EDIT
			// set the edit variable to the current saved value
			this.serverHostUrlForEdit = this.state.oauth_instance_url
			this.isOpenProjectInstanceValid = null
		},
		setNCOAuthFormToViewMode() {
			this.formMode.ncOauth = F_MODES.VIEW
			this.isFormCompleted.ncOauth = true
		},
		async saveOpenProjectHostUrl() {
			this.loadingServerHostForm = true
			await this.validateOpenProjectInstance()
			if (this.isOpenProjectInstanceValid) {
				const saved = await this.saveOPOptions()
				if (saved) {
					this.state.oauth_instance_url = this.serverHostUrlForEdit
					this.formMode.server = F_MODES.VIEW
					this.isFormCompleted.server = true
					if (!this.isFormCompleted.opOauth) {
						this.formMode.opOauth = F_MODES.EDIT
					}
				}
			}
			this.loadingServerHostForm = false
		},
		async saveOPOAuthClientValues() {
			await this.saveOPOptions()
			if (this.isAdminConfigOk) {
				this.formMode.opOauth = F_MODES.VIEW
				this.isFormCompleted.opOauth = true

				// if we do not have Nextcloud OAuth client yet, a new client is created
				if (!this.state.nc_oauth_client) {
					this.createNCOAuthClient()
				}
			}
		},
		resetOPOAuthClientValues() {
			OC.dialogs.confirmDestructive(
				t('integration_openproject', 'If you proceed you will need to update these settings with the new OpenProject OAuth credentials. Also, all users will need to reauthorize access to their OpenProject account.'),
				t('integration_openproject', 'Replace OpenProject OAuth values'),
				{
					type: OC.dialogs.YES_NO_BUTTONS,
					confirm: t('integration_openproject', 'Yes, replace'),
					confirmClasses: 'error',
					cancel: t('integration_openproject', 'Cancel'),
				},
				async (result) => {
					if (result) {
						await this.clearOPOAuthClientValues()
					}
				},
				true
			)
		},
		async clearOPOAuthClientValues() {
			this.formMode.opOauth = F_MODES.EDIT
			this.isFormCompleted.opOauth = false
			this.state.client_id = null
			this.state.client_secret = null
			const saved = await this.saveOPOptions()
			if (!saved) {
				this.formMode.opOauth = F_MODES.VIEW
				this.isFormCompleted.opOauth = true
			}
		},
		resetAllAppValuesConfirmation() {
			OC.dialogs.confirmDestructive(
				t('integration_openproject', 'Are you sure that you want to reset this app and delete all settings and all connections of all Nextcloud users to OpenProject?'),
				t('integration_openproject', 'Reset OpenProject integration'),
				{
					type: OC.dialogs.YES_NO_BUTTONS,
					confirm: t('integration_openproject', 'Yes, reset'),
					confirmClasses: 'error',
					cancel: t('integration_openproject', 'Cancel'),
				},
				async (result) => {
					if (result) {
						await this.resetAllAppValues()
					}
				},
				true
			)
		},
		async resetAllAppValues() {
			this.state.client_id = null
			this.state.client_secret = null
			this.state.oauth_instance_url = null
			await this.saveOPOptions()
			window.location.reload()
		},
		async validateOpenProjectInstance() {
			const url = generateUrl('/apps/integration_openproject/is-valid-op-instance')
			const response = await axios.post(url, { url: this.serverHostUrlForEdit })
			if (response.data === true) {
				this.isOpenProjectInstanceValid = true
				this.state.oauth_instance_url = this.serverHostUrlForEdit
			} else {
				if (response.data === 'invalid') {
					showError(
						t('integration_openproject', 'OpenProject URL is invalid, provide an URL in the form "https://openproject.org"')
					)
				} else {
					showError(
						t('integration_openproject', 'No OpenProject detected at the URL')
					)
				}
				this.isOpenProjectInstanceValid = false
				await this.$nextTick()
				await this.$refs['openproject-oauth-instance-input']?.$refs?.textInput?.focus()
			}
		},
		async saveOPOptions() {
			const url = generateUrl('/apps/integration_openproject/admin-config')
			const req = {
				values: {
					client_id: this.state.client_id,
					client_secret: this.state.client_secret,
					oauth_instance_url: this.state.oauth_instance_url,
				},
			}
			try {
				const response = await axios.put(url, req)
				// after successfully saving the admin credentials, the admin config status needs to be updated
				this.isAdminConfigOk = response?.data?.status === true
				showSuccess(t('integration_openproject', 'OpenProject admin options saved'))
				return true
			} catch (error) {
				console.debug(error)
				showError(
					t('integration_openproject', 'Failed to save OpenProject admin options')
				)
				return false
			}
		},
		resetNcOauthValues() {
			OC.dialogs.confirmDestructive(
				t('integration_openproject', 'If you proceed you will need to update the settings in your OpenProject with the new Nextcloud OAuth credentials. Also, all users in OpenProject will need to reauthorize access to their Nextcloud account.'),
				t('integration_openproject', 'Replace Nextcloud OAuth values'),
				{
					type: OC.dialogs.YES_NO_BUTTONS,
					confirm: t('integration_openproject', 'Yes, replace'),
					confirmClasses: 'error',
					cancel: t('integration_openproject', 'Cancel'),
				},
				async (result) => {
					if (result) {
						this.state.nc_oauth_client = null
						this.createNCOAuthClient()
					}
				},
				true
			)
		},
		createNCOAuthClient() {
			const url = generateUrl('/apps/integration_openproject/nc-oauth')
			axios.post(url).then((response) => {
				this.state.nc_oauth_client = response.data
				// generate part is complete but still the NC OAuth form is set to
				// edit mode and not completed state so that copy buttons will be available for the user
				this.formMode.ncOauth = F_MODES.EDIT
				this.isFormCompleted.ncOauth = false
			}).catch((error) => {
				showError(
					t('integration_openproject', 'Failed to create Nextcloud OAuth client')
					+ ': ' + error.response.request.responseText
				)
			})
		},
	},
}
</script>

<style scoped lang="scss">
@import '../../css/tab.css';

#reset-all-app-settings-btn {
	position: absolute;
	top: 30px;
	right: 22px;
}

#openproject_prefs {
	div {
		width: 100%;
	}
	.d-flex {
		display: flex;
		align-items: center;
	}
	.pb-1 {
		padding-bottom: .5rem;
	}
	.py-1 {
		padding: .3rem 0;
	}
	.mr-2 {
		margin-right: .5rem;
	}
}
</style>
<style lang="scss">
#openproject_prefs {
	.button-vue {
		height: 32px !important;
		min-height: 32px !important;

		&--vue-primary {
			.button-vue__text {
				color: #fff !important;
			}
		}

		&--vue-error {
			.button-vue__text {
				color: #fff !important;
			}
		}

		&__text {
			font-weight: 400 !important;
			font-size: 14px !important;
			line-height: 20px !important;
			color: #333333 !important;
		}
	}
	.form-actions {
		display: flex;
		align-items: center;
		padding: 15px 0;
	}
}

body.theme--dark, body[data-theme-dark], body[data-theme-dark-highcontrast] {
	#openproject_prefs {
		.button-vue {
			&__text {
				color: #fff !important;
			}
		}
	}
}
</style>
