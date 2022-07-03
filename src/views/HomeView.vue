<template>
  <IonPage>
    <IonHeader>
      <IonToolbar>
        <IonTitle>Biometric auth</IonTitle>
      </IonToolbar>
    </IonHeader>

    <IonContent
      :scroll-y="true"
      class="ion-padding-vertical"
    >
      <BiometryView />
    </IonContent>
  </IonPage>
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
