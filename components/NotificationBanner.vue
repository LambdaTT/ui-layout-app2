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
        <q-btn flat label="Sim" color="primary" @click="enableNotifications()"></q-btn>
        <q-btn flat label="Depois" color="primary" @click="showNotificationsBanner = false"></q-btn>
        <q-btn flat label="Nunca" color="primary" @click="neverShowNotificationsBanner()"></q-btn>
      </div>
    </q-card>
  </q-dialog>
</template>
<script>
// Services:
import { ENDPOINTS } from 'src/services/endpoints'

export default {
  name: 'ui-layoutapp2-notificationbanner',

  data() {
    return {
      isLogged: true,
      showNotificationsBanner: false,
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
    }
  },

  methods: {
    initNotificationsBanner() {
      // Checa se existe um item no local storage "Nunca exibir Notificações"
      let neverShow = localStorage.getItem('neverShowNotificationsBanner');
      
      // Caso 1: existe, retorna false
      if(neverShow) { this.showNotificationsBanner = false; } 
      
      // Caso 2: não existe, verifica permissões do usuário
      else {
        // Caso 2a: Usuário já definiu permissão
        if(Notification.permission === "granted" || Notification.permission === "denied") {
          this.showNotificationsBanner = false;
        }
        // Caso 2b: Usuário não definiu permissão
        else { this.showNotificationsBanner = true; }
      }
    },

    enableNotifications() {
      if(this.notificationsSupported) {
        Notification.requestPermission(result => {
          if(result == 'granted') {
            this.showNotificationsBanner = false;
            this.checkForExistingPushSubscription();
          } else if (result == 'denied') {
            this.showNotificationsBanner = false;
          }
        })
      }
    },

    neverShowNotificationsBanner() {
      localStorage.setItem('neverShowNotificationsBanner', true);
      this.showNotificationsBanner = false;
    },

    checkForExistingPushSubscription() {
      if(this.serviceWorkerSupported && this.notificationsSupported) {
        let reg;
        navigator.serviceWorker.ready.then(swreg => {
          reg = swreg;
          return swreg.pushManager.getSubscription()
        }).then(sub => {
          if(!sub) { this.createPushSubscription(reg) }
        }).catch(error => {
          console.error("Erro ao checar a inscrição push:", error);
        });
      }
    },

    createPushSubscription(reg) {
      let vapidPublicKey = '';
      let vapidPublicKeyConverted = this.urlBase64ToUint8Array(vapidPublicKey);

      reg.pushManager.subcribe({
        applicationServerKey: vapidPublicKeyConverted,
        userVisibleOnly: true
      }).then(newSub => {
        let newSubStringified = JSON.stringify(newSub),
            newSubJSON = JSON.parse(newSubStringified),
            newSubQueryString = this.$utils.objToSerialString(newSubJSON)

        console.log('newSub: ', newSub);
        console.log('JSON: ', newSubJSON);
        
        // Save subscription object in our DB
        // return this.$http.post(`${ENDPOINTS.PUSH}/createSubscription?${ newSubQueryString }`)
      }).catch(err => {
        console.log('err: ', err);
      })
    },

    urlBase64ToUint8Array(base64String) {
      const padding = '='.repeat((4 - base64String.length % 4) % 4);
      const base64 = (base64String + padding)
        .replace(/\-/g, '+')
        .replace(/_/g, '/');

      const rawData = window.atob(base64);
      const outputArray = new Uint8Array(rawData.length);

      for (let i = 0; i < rawData.length; ++i) {
        outputArray[i] = rawData.charCodeAt(i);
      }
      return outputArray;
    },
  },

  mounted() {
    this.initNotificationsBanner();
  },
}
</script>