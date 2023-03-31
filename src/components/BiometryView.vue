<template>
  <ion-list lines="full">
    <ion-list-header>Status</ion-list-header>

    <ion-item>
      {{ biometryDescription }}
    </ion-item>
  </ion-list>

  <ion-list
    class="mt-6"
    lines="full"
  >
    <ion-list-header>Options</ion-list-header>

    <ion-item v-if="!isNative">
      <!-- flex-initial puts the ion-select just after the label -->
      <ion-label class="flex-initial">Biometry:</ion-label>

      <ion-select
        v-model="biometryType"
        class="[--padding-start:0px] max-w-full"
        interface="action-sheet"
        :interface-options="{ header: 'Select biometry type' }"
        @ion-change="onSelectBiometry"
      >
        <ion-select-option
          v-for="entry in biometryTypes"
          :key="entry.type"
          :value="String(entry.type)"
        >
          {{ entry.title }}
        </ion-select-option>
      </ion-select>
    </ion-item>

    <ion-item v-if="isAndroid">
      <ion-label>Title:</ion-label>
      <ion-input
        v-model="options.androidTitle"
        type="text"
        autocapitalize="sentences"
      />
    </ion-item>

    <ion-item v-if="isAndroid">
      <ion-label>Subtitle:</ion-label>
      <ion-input
        v-model="options.androidSubtitle"
        type="text"
        autocapitalize="sentences"
      />
    </ion-item>

    <ion-item>
      <ion-label>Reason:</ion-label>
      <ion-input
        v-model="options.reason"
        type="text"
        autocapitalize="sentences"
      />
    </ion-item>

    <ion-item v-if="isNative">
      <ion-label>Cancel title:</ion-label>
      <ion-input
        v-model="options.cancelTitle"
        type="text"
        autocapitalize="sentences"
      />
    </ion-item>

    <ion-item v-if="isIOS">
      <ion-label>Fallback title:</ion-label>
      <ion-input
        v-model="options.iosFallbackTitle"
        type="text"
        autocapitalize="sentences"
      />
    </ion-item>

    <ion-item v-if="isNative">
      <ion-checkbox v-model="options.allowDeviceCredential" />
      <ion-label>Allow device credential</ion-label>
    </ion-item>
  </ion-list>

  <!-- We want to center the button -->
  <div class="flex justify-center">
    <ion-button
      class="mt-5"
      size="default"
      shape="round"
      @click="onAuthenticate"
    >
      Authenticate
    </ion-button>
  </div>
</template>

<script setup lang="ts">
import {
  type AuthenticateOptions,
  BiometricAuth,
  BiometryErrorType,
  BiometryType,
  type CheckBiometryResult,
  getBiometryName,
  type ResultError
} from '@aparajita/capacitor-biometric-auth'
import { Capacitor, type PluginListenerHandle } from '@capacitor/core'
import type { SelectCustomEvent } from '@ionic/core'
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
import {
  computed,
  onBeforeMount,
  onBeforeUnmount,
  reactive,
  ref,
  toRaw
} from 'vue'

const biometryTypes = [
  {
    title: 'None',
    type: BiometryType.none
  },
  {
    title: 'Touch ID',
    type: BiometryType.touchId
  },
  {
    title: 'Face ID',
    type: BiometryType.faceId
  },
  {
    title: 'Fingerprint Authentication',
    type: BiometryType.fingerprintAuthentication
  },
  {
    title: 'Face Authentication',
    type: BiometryType.faceAuthentication
  },
  {
    title: 'Iris Authentication',
    type: BiometryType.irisAuthentication
  }
]
/*
 * ref
 */
const biometry = ref<CheckBiometryResult>({
  isAvailable: false,
  biometryType: BiometryType.none,
  reason: '',
  code: BiometryErrorType.none
})

const options = reactive<AuthenticateOptions>({
  reason: '',
  cancelTitle: '',
  iosFallbackTitle: '',
  androidTitle: '',
  androidSubtitle: '',
  allowDeviceCredential: false
})

const biometryType = ref(String(BiometryType.none))
const appListener = ref<PluginListenerHandle>()

/*
 * computed
 */
const biometryName = computed(() => {
  return getBiometryName(biometry.value.biometryType) || 'No biometry'
})

const biometryDescription = computed(() => {
  let description = `${biometryName.value} is supported`

  if (biometry.value.biometryType !== BiometryType.none) {
    if (biometry.value.isAvailable) {
      description += ' and available.'
    } else {
      description += ', but not available.'
    }

    if (biometry.value.reason) {
      description += ` ${biometry.value.reason} (${biometry.value.code})`
    }
  } else {
    description += '.'
  }

  return description
})

const isNative = computed(() => Capacitor.isNativePlatform())

const isIOS = computed(() => Capacitor.isNativePlatform() && isPlatform('ios'))

const isAndroid = computed(
  () => Capacitor.isNativePlatform() && isPlatform('android')
)

/*
 * methods
 */
function updateBiometryInfo(info: CheckBiometryResult): void {
  biometry.value = info
}

onBeforeMount(async () => {
  updateBiometryInfo(await BiometricAuth.checkBiometry())

  try {
    appListener.value = await BiometricAuth.addResumeListener(
      updateBiometryInfo
    )
  } catch (error) {
    if (error instanceof Error) {
      console.error(error.message)
    }
  }
})

onBeforeUnmount(async () => {
  await appListener.value?.remove()
})

async function showAlert(message: string): Promise<void> {
  const alert = await alertController.create({
    header: `${biometryName.value} says:`,
    subHeader: '',
    message,
    buttons: ['OK']
  })
  await alert.present()
}

async function showErrorAlert(error: ResultError): Promise<void> {
  await showAlert(`${error.message} [${error.code}].`)
}

async function onAuthenticate(): Promise<void> {
  try {
    // options is a reactive proxy, we can't pass it directly to a plugin.
    // so pass the underlying object.
    await BiometricAuth.authenticate(toRaw(options))
    await showAlert('Authorization successful!')
  } catch (error) {
    // Someday TypeScript will let us type catch clauses...
    // eslint-disable-next-line @typescript-eslint/consistent-type-assertions
    await showErrorAlert(error as ResultError)
  }
}

async function onSelectBiometry(
  event: SelectCustomEvent<string>
): Promise<void> {
  await BiometricAuth.setBiometryType(Number(event.detail.value))
  updateBiometryInfo(await BiometricAuth.checkBiometry())
}
</script>
