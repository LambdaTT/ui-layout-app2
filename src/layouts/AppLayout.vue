<template>
  <div class="text-grey-9" style="background: transparent">
    <q-layout view="hHh Lpr lFf">
      <Header
        @toggleDrawer="drawerState = !drawerState"
        :Actions="HeaderOptions"
        :MainLogoSrc="logo"
        :HeaderStyle="HeaderStyle"
      >
        <template #header-options>
          <NotificationBell v-if="isLogged"></NotificationBell>
          <!-- Slot para o pai injetar ações específicas do app -->
          <slot name="header-actions"></slot>
        </template>
      </Header>

      <Sidebar
        :NavItems="NavItems"
        :Socials="socials"
        :MainLogoSrc="logo"
        :SidebarStyle="SidebarStyle"
        v-model="drawerState"
      >
        <template #footer v-if="'sidebar-footer' in $slots">
          <slot name="sidebar-footer"></slot>
        </template>
      </Sidebar>

      <q-page-container>
        <div id="content-wrapper" :style="ContentStyle">
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

export default {
  props: {
    HeaderOptions: Array,
    NavItems: {
      type: Array,
      default: () => [],
    },
    DefaultRoute: {
      type: String,
      default: () => "/",
    },
    HeaderStyle: [Object, String],
    SidebarStyle: [Object, String],
    ContentStyle: [Object, String],
  },

  data() {
    return {
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

    currentRoute() {
      return this.$route.fullPath;
    },
  },

  watch: {
    currentRoute(v) {
      if (v === "/" || v === "") this.$router.push(this.DefaultRoute);
    },
  },

  methods: {
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
        .get(`${ENDPOINTS.SETTINGS.SINGLE}/institutional/logo`)
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
  },

  async mounted() {
    if (await this.isInMaintenance()) {
      this.$router.push("/maintenance");
      return;
    }

    if (this.$route.fullPath === "/" || this.$route.fullPath === "") {
      this.$router.push(this.DefaultRoute);
      return;
    }

    await this.getLogo();
    await this.getSocials();
    this.inactivityHandler();
  },
};
</script>
<style scoped>
#content-wrapper {
  text-align: center;
}
</style>
