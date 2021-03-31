<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>
          Biometric auth
        </ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content
      scroll-y="true"
      class="ion-padding-vertical"
    >
      <Biometry />
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import {
  IonContent,
  IonHeader,
  IonPage,
  IonTitle,
  IonToolbar
} from '@ionic/vue'
import Biometry from '@/components/Biometry.vue'

export default {
  components: {
    Biometry,
    IonContent,
    IonHeader,
    IonPage,
    IonTitle,
    IonToolbar
  }
}
</script>
