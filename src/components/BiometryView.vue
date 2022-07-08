<template>
  <IonList lines="full">
    <IonListHeader>Status</IonListHeader>

    <IonItem>
      {{ biometryDescription }}
    </IonItem>
  </IonList>

  <IonList
    class="mt-6"
    lines="full"
  >
    <IonListHeader>Options</IonListHeader>

    <IonItem v-if="!isNative">
      <!-- flex-initial puts the IonSelect just after the label -->
      <IonLabel class="flex-initial">Biometry:</IonLabel>

      <IonSelect
        v-model="biometryType"
        class="[--padding-start:0px] max-w-full"
        interface="action-sheet"
        :interface-options="{ header: 'Select biometry type' }"
        @ion-change="onSelectBiometry"
      >
        <IonSelectOption
          v-for="entry in biometryTypes"
          :key="entry.type"
          :value="String(entry.type)"
        >
          {{ entry.title }}
        </IonSelectOption>
      </IonSelect>
    </IonItem>

    <IonItem v-if="isAndroid">
      <IonLabel>Title:</IonLabel>
      <IonInput
        v-model="options.androidTitle"
        type="text"
        autocapitalize="sentences"
      />
    </IonItem>

    <IonItem v-if="isAndroid">
      <IonLabel>Subtitle:</IonLabel>
      <IonInput
        v-model="options.androidSubtitle"
        type="text"
        autocapitalize="sentences"
      />
    </IonItem>

    <IonItem>
      <IonLabel>Reason:</IonLabel>
      <IonInput
        v-model="options.reason"
        type="text"
        autocapitalize="sentences"
      />
    </IonItem>

    <IonItem v-if="isNative">
      <IonLabel>Cancel title:</IonLabel>
      <IonInput
        v-model="options.cancelTitle"
        type="text"
        autocapitalize="sentences"
      />
    </IonItem>

    <IonItem v-if="isIOS">
      <IonLabel>Fallback title:</IonLabel>
      <IonInput
        v-model="options.iosFallbackTitle"
        type="text"
        autocapitalize="sentences"
      />
    </IonItem>

    <IonItem v-if="isAndroid">
      <IonLabel>Max attempts:</IonLabel>
      <IonInput
        v-model.number="options.androidMaxAttempts"
        type="number"
        inputmode="decimal"
        min="1"
        max="10"
      />
    </IonItem>

    <IonItem v-if="isNative">
      <IonCheckbox v-model="options.allowDeviceCredential" />
      <IonLabel>Allow device credential</IonLabel>
    </IonItem>
  </IonList>

  <!-- We want to center the button -->
  <div class="flex justify-center">
    <IonButton
      class="mt-5"
      size="default"
      shape="round"
      @click="onAuthenticate"
    >
      Authenticate
    </IonButton>
  </div>
</template>

<script setup lang="ts">
import {
  type AuthenticateOptions,
  BiometricAuth,
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
      description += ` ${biometry.value.reason}`
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
  console.log(info)
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
  BiometricAuth.setBiometryType(Number(event.detail.value))
  updateBiometryInfo(await BiometricAuth.checkBiometry())
}
</script>
