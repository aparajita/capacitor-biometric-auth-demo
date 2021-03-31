<template>
  <div class="flex flex-col justify-start items-center w-full h-full py-1">
    <p class="px-4 text-center">
      {{ biometryDescription }}
    </p>

    <div class="flex flex-col justify-start items-start flex-1 w-full">
      <ion-list class="w-full">
        <ion-list-header>Options</ion-list-header>

        <ion-item
          v-if="!isNative"
          lines="none"
        >
          <ion-label class="flex-initial">
            Biometry:
          </ion-label>

          <ion-select
            v-model="biometryType"
            interface="action-sheet"
            :interface-options="{ header: 'Select biometry type' }"
            class="max-w-none pl-0 "
            @ionChange="onSelectBiometry"
          >
            <ion-select-option value="none">
              None
            </ion-select-option>

            <ion-select-option
              value="touchId"
            >
              Touch ID
            </ion-select-option>

            <ion-select-option
              value="faceId"
            >
              Face ID
            </ion-select-option>

            <ion-select-option
              value="fingerprintAuthentication"
            >
              Fingerprint Authentication
            </ion-select-option>

            <ion-select-option
              value="faceAuthentication"
            >
              Face Authentication
            </ion-select-option>

            <ion-select-option
              value="irisAuthentication"
            >
              Iris Authentication
            </ion-select-option>
          </ion-select>
        </ion-item>

        <ion-item lines="none">
          <ion-label>
            Reason:
          </ion-label>
          <ion-input
            v-model="options.reason"
            type="text"
            autocapitalize="sentences"
          />
        </ion-item>

        <ion-item
          v-if="isNative"
          lines="none"
        >
          <ion-label>
            Cancel title:
          </ion-label>
          <ion-input
            v-model="options.cancelTitle"
            type="text"
            autocapitalize="sentences"
          />
        </ion-item>

        <ion-item
          v-if="isIOS"
          lines="none"
        >
          <ion-label>
            Fallback title:
          </ion-label>
          <ion-input
            v-model="options.iosFallbackTitle"
            type="text"
            autocapitalize="sentences"
          />
        </ion-item>

        <ion-item
          v-if="isAndroid"
          lines="none"
        >
          <ion-label>
            Title:
          </ion-label>
          <ion-input
            v-model="options.androidTitle"
            type="text"
            autocapitalize="sentences"
          />
        </ion-item>

        <ion-item
          v-if="isAndroid"
          lines="none"
        >
          <ion-label>
            Subtitle:
          </ion-label>
          <ion-input
            v-model="options.androidSubtitle"
            type="text"
            autocapitalize="sentences"
          />
        </ion-item>

        <ion-item
          v-if="isAndroid"
          lines="none"
        >
          <ion-label>
            Max attempts:
          </ion-label>
          <ion-input
            v-model.number="options.androidMaxAttempts"
            type="number"
            inputmode="decimal"
            min="1"
            max="10"
          />
        </ion-item>

        <ion-item
          v-if="isNative"
          lines="none"
        >
          <ion-checkbox
            v-model="options.allowDeviceCredential"
          />
          <ion-label class="text-base">
            Allow device credential
          </ion-label>
        </ion-item>
      </ion-list>

      <ion-button
        class="mt-4 mx-auto"
        @click="onAuthenticate"
      >
        Authenticate
      </ion-button>
    </div>
  </div>
</template>

<script lang="ts">
import {
  alertController,
  IonButton,
  IonCheckbox,
  IonInput,
  IonItem,
  IonLabel,
  IonList,
  IonListHeader,
  IonSelect,
  IonSelectOption,
  isPlatform
} from '@ionic/vue'
import { Capacitor, Plugins } from '@capacitor/core'
import {
  AuthenticateOptions,
  BiometryType,
  CheckBiometryResult,
  getBiometryName,
  ResultError,
  WSBiometricAuthPlugin
} from '@aparajita/capacitor-biometric-auth'
import { computed, onBeforeMount, reactive, ref } from 'vue'
import { SelectChangeEventDetail } from '@ionic/core'

const auth = Plugins.WSBiometricAuth as WSBiometricAuthPlugin

export default {
  components: {
    IonButton,
    IonCheckbox,
    IonInput,
    IonItem,
    IonLabel,
    IonList,
    IonListHeader,
    IonSelect,
    IonSelectOption
  },

  setup() {
    /*
     * ref
     */
    const biometry = ref<CheckBiometryResult>({
      isAvailable: false,
      biometryType: BiometryType.none,
      reason: ''
    })

    const options = reactive<AuthenticateOptions>({
      reason: '',
      cancelTitle: '',
      iosFallbackTitle: '',
      androidTitle: '',
      androidSubtitle: '',
      allowDeviceCredential: false,
      androidMaxAttempts: 3
    })

    const biometryType = ref('none')

    /*
     * computed
     */
    const biometryName = computed(
      () => getBiometryName(biometry.value.biometryType) || 'No biometry'
    )

    const biometryDescription = computed(() => {
      let description = `${biometryName.value} is supported`

      if (biometry.value.biometryType !== BiometryType.none) {
        if (biometry.value.isAvailable) {
          description += ' and available.'
        } else {
          description += ', but not available.'
        }

        if (biometry.value.reason) {
          description += ` ${biometry.value.reason}`
        }
      }

      return description
    })

    const isNative = computed(() => Capacitor.isNative)

    const isIOS = computed(() => Capacitor.isNative && isPlatform('ios'))

    const isAndroid = computed(
      () => Capacitor.isNative && isPlatform('android')
    )

    /*
     * methods
     */
    function updateBiometryInfo(info: CheckBiometryResult) {
      biometry.value = info
    }

    onBeforeMount(async () => {
      updateBiometryInfo(await auth.checkBiometry())
      auth.addResumeListener(updateBiometryInfo)
    })

    async function onAuthenticate() {
      try {
        await auth.authenticate({
          reason: options.reason,
          cancelTitle: options.cancelTitle,
          iosFallbackTitle: options.iosFallbackTitle,
          androidTitle: options.androidTitle,
          androidSubtitle: options.androidSubtitle,
          allowDeviceCredential: options.allowDeviceCredential,
          androidMaxAttempts: options.androidMaxAttempts
        })
        await showAlert('Authorization successful!')
      } catch (e) {
        await showErrorAlert(e)
      }
    }

    async function onSelectBiometry(
      event: CustomEvent<SelectChangeEventDetail>
    ) {
      auth.setBiometryType(event.detail.value)
      updateBiometryInfo(await auth.checkBiometry())
    }

    async function showErrorAlert(error: ResultError) {
      await showAlert(`${error.message} [${error.code}].`)
    }

    async function showAlert(message: string) {
      const alert = await alertController.create({
        header: `${biometryName.value} says:`,
        subHeader: '',
        message,
        buttons: ['OK']
      })
      await alert.present()
    }

    return {
      biometryDescription,
      biometryType,
      isAndroid,
      isIOS,
      isNative,
      onAuthenticate,
      onSelectBiometry,
      options
    }
  }
}
</script>
