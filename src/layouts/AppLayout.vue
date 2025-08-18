<template>
  <div class="text-grey-9">
    <q-layout view="hHh Lpr lFf" container-fluid style="height: 300px" class="shadow-2 rounded-borders">
      <Header @toggleDrawer="drawerState = !drawerState;" :Actions="headerActions" :MainLogoSrc="logo"
        BtnActionsIcon="fas fa-user" :LoadNotificationsFn="loadNotifications" :ShowNotification="isLogged"></Header>

      <Sidebar :NavItems="navItems" :Socials="socials" :MainLogoSrc="logo" @load="load" @loaded="loaded"
        v-model="drawerState" />

      <q-page-container>
        <div id="content-wrapper">
          <InstallationBanner class="q-px-sm"></InstallationBanner>
          <div id="page-wrapper">
            <router-view @load="load" @loaded="loaded" />
          </div>
          <PushNotifications></PushNotifications>
        </div>
      </q-page-container>
    </q-layout>
  </div>
</template>

<script>
import ENDPOINTS from '../ENDPOINTS'

// Libs:
import { useQuasar } from 'quasar'

export default {
  data() {
    return {
      $q: useQuasar(),
      isLogged: false,
      toLoad: [],
      drawerState: false,
      socials: {},
      logo: null,
      notificationsDialog: true,
    }
  },

  computed: {
    headerActions() {
      if (this.isLogged) {
        return [
          { title: 'Meu Perfil', icon: 'fas fa-user', fn: () => this.$router.push('/myaccount') },
          { title: 'Meu Portal', icon: 'fas fa-th-large', fn: () => this.$router.push('/loggedarea') },
          { title: 'Sair', icon: 'fas fa-sign-out', fn: this.logout }
        ];
      } else {
        return [
          { title: 'Entrar', icon: 'fas fa-sign-in', fn: () => this.$router.push('/login') },
        ];
      }
    },

    navItems() {
      let items = [
        { title: 'Página Inicial', icon: 'fas fa-home', fn: () => this.$router.push('/') },
      ]
      return items;
    }
  },

  methods: {
    load(evt) {
      this.$q.loading.show();
      if (evt && evt != '')
        this.toLoad.push(evt);
    },

    loaded(evt) {
      var index = this.toLoad.indexOf(evt);

      if (index != -1) this.toLoad.splice(index, 1);

      if (this.toLoad.length == 0) {
        this.$q.loading.hide();
      }
    },

    async loadNotifications() {
      try {
        // Emitting the loading event
        this.$emit('load', 'notifications-load');
        // API request
        const response = await this.$getService('toolcase/http').get(ENDPOINTS.NOTIFICATIONS.COUNT);
        if (response && response.data) { return response.data; }
      } catch (error) {
        if (error.response?.status != 401)
          this.$utils.notifyError(error);

        console.error("An error occurred while attempting to retrieve notifications.", error);
      } finally {
        // Finalizing the loading event
        this.$emit('loaded', 'notifications-load');
      }
    },

    async logout() {
      if (!confirm("Deseja encerrar seu acesso?")) return false;
      await this.$getService('iam/auth').logout()
      this.$router.push('/');
    },

    async getSocials() {
      // Emitting the loading event
      this.$emit('load', 'socials-read');
      try {
        const response = await this.$getService('toolcase/http').get(`${ENDPOINTS.SETTINGS.CONTEXT_OBJ}/socials`);
        if (response && response.data) {
          this.socials = response.data.reduce((acc, item) => {
            if (item.tx_fieldvalue == null) return acc;
            acc[item.ds_fieldname.split('_')[0]] = item.tx_fieldvalue;
            return acc;
          }, {});
        }
      } catch (error) {
        this.$utils.notifyError(error);
        console.error("An error occurred while attempting to retrieve the object's data.", error);
      } finally {
        // Finalizing the loading event
        this.$emit('loaded', 'channel-read');
      }
    },

    getLogo() {
      return this.$getService('toolcase/http').get(`${ENDPOINTS.SETTINGS.SINGLE}/institution/logo`)
        .then((response) => {
          this.logo = response.data.tx_fieldvalue || '/resources/img/logo-sindicato.png'
        })
    },

    async isInMaintenance() {
      try {
        const response = await this.$getService('toolcase/http').get(`${ENDPOINTS.SETTINGS.SINGLE}/general/isInMaintenance`);
        if (response && response.data) {
          return response.data.tx_fieldvalue === "Y";
        };
      } catch (error) {
        this.$getService('toolcase/utils').notifyError(error);
        console.error("An error occurred while attempting to retrieve the object's data.", error);
      }
    },

    inactivityHandler() {
      var debounceTimeout = null;

      this.$eventbroadcaster.$on('http-request-sent', (reqPromise) => {
        reqPromise.catch((err) => {
          if (!!debounceTimeout) {
            clearTimeout(debounceTimeout);
            debounceTimeout = null;
          }
          debounceTimeout = setTimeout(() => {
            if (err.response?.status == 401 && !(err.config?.url.includes('/iam/auth/v1/log'))) {
              this.$router.go(0);

              this.$utils.notify({
                message: 'Sua sessão expirou. Por favor, entre novamente.',
                type: 'warning',
                position: 'top-right',
              });
            }
          }, 200);
        });
      });
    },

    loadHandler() {
      this.$eventbroadcaster.$on('load', this.load);
      this.$eventbroadcaster.$on('loaded', this.loaded);
    }
  },

  async mounted() {
    if (await this.isInMaintenance()) this.$router.push('/maintenance');
    await this.getLogo();
    await this.getSocials();

    this.isLogged = await this.$getService('iam/auth').authenticate();
    this.inactivityHandler();
    this.loadHandler();
  },
}
</script>
<style scoped>
#content-wrapper {
  background-color: #e9ecef;
  text-align: center;
}
</style>
