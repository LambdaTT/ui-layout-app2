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
        <q-btn flat label="Sim" color="primary" @click="() => enableNotifications()"></q-btn>
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

    async enableNotifications() {
      try {
        this.showNotificationsBanner = false;
        const token = await Firebase.requestPermission();
        console.log("Token FCM:\n",token);
        
        await this.$http.post(ENDPOINTS.PUSH + '/subscription', {token: token});
      } catch (error) {
        console.error('Error requesting notification permission.', error);
      }
    },
  },

  async mounted() {
    this.showNotificationsBanner = this.initNotificationsBanner() && await Auth.authenticate();
  },
}
</script>