<template>
  <div class="text-grey-9" style="background: transparent">
    <q-layout view="hHh Lpr lFf">
      <Header
        :ShowSidebarBtn="NavItems.length"
        @toggleDrawer="drawerState = !drawerState"
        :Actions="HeaderOptions"
        :MainLogoSrc="logo"
        :HeaderStyle="HeaderStyle"
      >
        <template #header-options>
          <!-- Slot para o pai injetar ações específicas do app -->
          <slot name="header-actions"></slot>
        </template>
      </Header>

      <Sidebar
        v-if="NavItems.length"
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
          <PushNotification v-if="'push-notification' in $slots">
            <slot name="push-notification"> </slot>
          </PushNotification>
          <InstallationBanner v-if="Installable" class="q-px-sm"></InstallationBanner>
          <div id="page-wrapper">
            <router-view @load="load" @loaded="loaded" />
          </div>
        </div>
      </q-page-container>
    </q-layout>
  </div>
</template>

<script>
import ENDPOINTS from '../ENDPOINTS'

export default {
  props: {
    HeaderOptions: Array,
    NavItems: {
      type: Array,
      default: () => [],
    },
    DefaultRoute: {
      type: String,
      default: () => '/',
    },
    HeaderStyle: [Object, String],
    SidebarStyle: [Object, String],
    ContentStyle: [Object, String],
    Installable: Boolean,
  },

  data() {
    return {
      drawerState: false,
      socials: {},
      logo: null,
      notificationsDialog: true,
    }
  },

  computed: {
    currentRoute() {
      return this.$route.fullPath
    },
  },

  watch: {
    currentRoute(v) {
      if (v === '/' || v === '') this.$router.push(this.DefaultRoute)
    },
  },

  methods: {
    async getSocials() {
      // Emitting the loading event
      this.$emit('load', 'socials-read')
      try {
        const response = await this.$getService('toolcase/http').get(
          `${ENDPOINTS.SETTINGS.CONTEXT_OBJ}/socials`,
        )
        if (response && response.data) {
          this.socials = Object.keys(response.data).reduce((obj, key) => {
            if (response.data[key]) obj[key.split('_')[0]] = response.data[key]
            return obj
          }, {})
        }
      } catch (error) {
        if (error.status !== 404) {
          this.socials = {}
          this.$getService('toolcase/utils').notifyError(error)
          console.error("An error occurred while attempting to retrieve the object's data.", error)
        }
      } finally {
        // Finalizing the loading event
        this.$emit('loaded', 'channel-read')
      }
    },

    getLogo() {
      return this.$getService('toolcase/http')
        .get(`${ENDPOINTS.SETTINGS.SINGLE}/institutional/logo`)
        .then((response) => {
          this.logo = response.data.tx_fieldvalue || '/resources/img/logo-sindicato.png'
        })
    },

    async isInMaintenance() {
      try {
        const response = await this.$getService('toolcase/http').get(
          `${ENDPOINTS.SETTINGS.SINGLE}/general/isInMaintenance`,
        )
        if (response && response.data) {
          return response.data.tx_fieldvalue === 'Y'
        }
      } catch (error) {
        this.$getService('toolcase/utils').notifyError(error)
        console.error("An error occurred while attempting to retrieve the object's data.", error)
      }
    },
  },

  async mounted() {
    if (await this.isInMaintenance()) {
      this.$router.push('/maintenance')
      return
    }

    // Dados do layout (logo do estabelecimento, redes sociais) precisam carregar
    // em QUALQUER rota — inclusive na raiz '/', que é a rota do cardápio. O early
    // return anterior, quando fullPath === '/', pulava o getLogo() e deixava o
    // topo sem o logo configurado.
    await this.getLogo()
    await this.getSocials()

    if (this.$route.fullPath === '/' || this.$route.fullPath === '') {
      this.$router.push(this.DefaultRoute)
    }
  },
}
</script>
<style scoped>
#content-wrapper {
  text-align: center;
}
</style>
