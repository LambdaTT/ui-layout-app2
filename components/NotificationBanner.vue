<template>
  <q-dialog v-model="showNotificationsBanner">
    <q-card flat class="floating-card q-pa-sm">
      <div class="row q-pa-sm">
        <div class="col-auto">
          <q-icon name="far fa-bell" color="primary" size="sm"></q-icon>
        </div>
        <span class="col q-ml-md text-left">Você gostaria de ativar as notificações?</span>
      </div>
      <div class="row justify-end">
        <q-btn flat label="Sim" color="primary" @click="requestPushPermission()"></q-btn>
        <q-btn flat label="Depois" color="primary" @click="showNotificationsBanner = false"></q-btn>
        <q-btn flat label="Nunca" color="primary" @click="neverShowNotificationsBanner()"></q-btn>
      </div>
    </q-card>
  </q-dialog>
</template>
<script>
// Services:
import { ENDPOINTS } from 'src/services/endpoints'
import Auth from 'src/services/auth'
import Firebase from 'src/services/firebase'

const vapidPublicKey = 'BDdP-T2uVLZkytvcZ3cy7zls0LTdwRkMcsU-wDELTTmCX_PqCIc7Y5MXG9Qqau1RFOBJFJvy5lqaSITspzIgEyI';

export default {
  name: 'ui-layoutapp2-notificationbanner',

  data() {
    return {
      showNotificationsBanner: null,
    }
  },

  computed: {
    notificationsSupported() {
      if ("Notification" in window) { return true; }
      return false;
    },

    serviceWorkerSupported() {
      if ("serviceWorker" in navigator) { return true; }
      return false;
    },
  },

  methods: {
    initNotificationsBanner() {
      if(this.notificationsSupported && this.serviceWorkerSupported) {
        const neverShow = !!localStorage.getItem('neverShowNotificationsBanner')
        const permissionSelected = Notification.permission === "granted" || Notification.permission === "denied";
        return !(neverShow || permissionSelected);
      }
      return false;
    },

    neverShowNotificationsBanner() {
      localStorage.setItem('neverShowNotificationsBanner', true);
      this.showNotificationsBanner = false;
    },

    async requestPushPermission() {
      try {
        this.showNotificationsBanner = false;
        const result = await Notification.requestPermission();
        if(result === 'granted') {
          console.log('Notification permission granted.');
          await this.checkForExistingPushSubscription();
        }
      } catch (error) {
        console.error('Error requesting notification permission.', error);
      }
    },

    async checkForExistingPushSubscription() {
      try {
        const reg = await navigator.serviceWorker.ready;
        const sub = await reg.pushManager.getSubscription();
        if(!sub) { this.createPushSubscription(reg); }
      } catch (error) {
        console.error("Error checking push subscription.", error);
      }
    },

    async createPushSubscription(reg) {
      let vapidPublicKeyConverted = this.urlBase64ToUint8Array(vapidPublicKey);

      // Create push subscription
      const subscription = await reg.pushManager.subscribe({
        userVisibleOnly: true,
        applicationServerKey: vapidPublicKeyConverted,
      });
      console.log('Push subscription created.');
      
      // Save subscription object in our DB
      const subscriptionJSON = JSON.parse(JSON.stringify(subscription));
      subscriptionJSON.token = await Firebase.getFCMToken(vapidPublicKey);
      await this.$http.post(ENDPOINTS.PUSH + '/subscription', subscriptionJSON);
      console.log('Push subscription saved in DB', subscriptionJSON);
    },
  
    urlBase64ToUint8Array(base64String) {
      const padding = '='.repeat((4 - base64String.length % 4) % 4);
      const base64 = (base64String + padding)
        .replace(/\-/g, '+')
        .replace(/_/g, '/');

      const rawData = atob(base64);
      const outputArray = new Uint8Array(rawData.length);

      for (let i = 0; i < rawData.length; i++) {
        outputArray[i] = rawData.charCodeAt(i);
      }
      return outputArray;
    },
  },

  async mounted() {
    this.showNotificationsBanner = this.initNotificationsBanner() && await Auth.authenticate();
  },
}
</script>