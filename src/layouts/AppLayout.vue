<template>
  <div class="text-grey-9">
    <q-layout
      view="hHh Lpr lFf"
      container-fluid
      style="height: 300px"
      class="shadow-2 rounded-borders"
    >
      <Header
        @toggleDrawer="drawerState = !drawerState"
        :Actions="HeaderOptions"
        :MainLogoSrc="logo"
        BtnActionsIcon="fas fa-user"
      >
        <template #header-options>
          <NotificationBell v-if="isLogged"></NotificationBell>
        </template>
      </Header>

      <Sidebar
        :NavItems="NavItems"
        :Socials="socials"
        :MainLogoSrc="logo"
        @load="load"
        @loaded="loaded"
        v-model="drawerState"
      />

      <q-page-container>
        <div id="content-wrapper">
          <PushNotification>
            <div class="row">
              <div class="col-auto q-mb-md">
                <q-icon name="far fa-bell" color="grey-9" size="sm"></q-icon>
              </div>
              <span>
                Para manter-se atualizado com as últimas notícias habilite as
                notificações.
              </span>
            </div>
          </PushNotification>
          <InstallationBanner class="q-px-sm"></InstallationBanner>
          <div id="page-wrapper">
            <router-view @load="load" @loaded="loaded" />
          </div>
        </div>
      </q-page-container>
    </q-layout>
  </div>
</template>

<script>
import ENDPOINTS from "../ENDPOINTS";

// Libs:
import { useQuasar } from "quasar";

export default {
  props: {
    HeaderOptions: Array,
    NavItems: {
      type: Array,
      default: () => [],
    },
  },

  data() {
    return {
      $q: useQuasar(),
      toLoad: [],
      drawerState: false,
      socials: {},
      logo: null,
      notificationsDialog: true,

      // Modules:
      $msg: this.$getModule("messaging"),
      $iam: this.$getModule("iam"),
    };
  },

  computed: {
    isLogged() {
      return !!this.$getService("iam/auth").getLoggedUser();
    },
  },

  methods: {
    load(evt) {
      this.$q.loading.show();
      if (evt && evt != "") this.toLoad.push(evt);
    },

    loaded(evt) {
      var index = this.toLoad.indexOf(evt);

      if (index != -1) this.toLoad.splice(index, 1);

      if (this.toLoad.length == 0) {
        this.$q.loading.hide();
      }
    },

    async logout() {
      if (!confirm("Deseja encerrar seu acesso?")) return false;
      await this.$getService("iam/auth").logout();
      this.$router.push("/login");
    },

    async getSocials() {
      // Emitting the loading event
      this.$emit("load", "socials-read");
      try {
        const response = await this.$getService("toolcase/http").get(
          `${ENDPOINTS.SETTINGS.CONTEXT_OBJ}/socials`,
        );
        if (response && response.data) {
          this.socials = Object.keys(response.data).reduce((obj, key) => {
            if (response.data[key]) obj[key.split("_")[0]] = response.data[key];
            return obj;
          }, {});
        }
      } catch (error) {
        if (error.status !== 404) {
          this.socials = {};
          this.$getService("toolcase/utils").notifyError(error);
          console.error(
            "An error occurred while attempting to retrieve the object's data.",
            error,
          );
        }
      } finally {
        // Finalizing the loading event
        this.$emit("loaded", "channel-read");
      }
    },

    getLogo() {
      return this.$getService("toolcase/http")
        .get(`${ENDPOINTS.SETTINGS.SINGLE}/institution/logo`)
        .then((response) => {
          this.logo =
            response.data.tx_fieldvalue || "/resources/img/logo-sindicato.png";
        });
    },

    async isInMaintenance() {
      try {
        const response = await this.$getService("toolcase/http").get(
          `${ENDPOINTS.SETTINGS.SINGLE}/general/isInMaintenance`,
        );
        if (response && response.data) {
          return response.data.tx_fieldvalue === "Y";
        }
      } catch (error) {
        this.$getService("toolcase/utils").notifyError(error);
        console.error(
          "An error occurred while attempting to retrieve the object's data.",
          error,
        );
      }
    },

    inactivityHandler() {
      var debounceTimeout = null;

      this.$getService("toolcase/eventbroadcaster").$on(
        "http-request-sent",
        (reqPromise) => {
          reqPromise.catch((err) => {
            if (!!debounceTimeout) {
              clearTimeout(debounceTimeout);
              debounceTimeout = null;
            }
            debounceTimeout = setTimeout(() => {
              if (
                err.response?.status == 401 &&
                !err.config?.url.includes("/iam/auth/v1/log")
              ) {
                this.$router.go(0);

                this.$getService("toolcase/utils").notify({
                  message: "Sua sessão expirou. Por favor, entre novamente.",
                  type: "warning",
                  position: "top-right",
                });
              }
            }, 200);
          });
        },
      );
    },

    loadHandler() {
      this.$getService("toolcase/eventbroadcaster").$on("load", this.load);
      this.$getService("toolcase/eventbroadcaster").$on("loaded", this.loaded);
    },
  },

  async mounted() {
    this.$q.loading.show();
    if (await this.isInMaintenance()) {
      this.$router.push("/maintenance");
      return;
    }

    await this.getLogo();
    await this.getSocials();
    this.inactivityHandler();
    this.loadHandler();
    this.$q.loading.hide();
  },
};
</script>
<style scoped>
#content-wrapper {
  background-color: #e9ecef;
  text-align: center;
}
</style>
