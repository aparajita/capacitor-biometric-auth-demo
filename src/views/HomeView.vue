<template>
  <ion-page class="w-full h-full">
    <ion-header>
      <ion-toolbar>
        <ion-title>Biometric auth</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content
      :scroll-y="true"
      class="ion-padding-vertical"
    >
      <BiometryView />
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { SplashScreen } from '@capacitor/splash-screen'
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar
} from '@ionic/vue'
import { onMounted } from 'vue'
import BiometryView from '@/components/BiometryView.vue'

onMounted(async (): Promise<void> => {
  await SplashScreen.hide()
})
</script>
