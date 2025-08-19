<template>
  <div class="column bg-grey-3" style="height: 100%;">
    <q-card square>
      <q-toolbar :class="`text-grey-9 ${logo ? 'menu' : 'bg-grey-10'}`" style="height: 89px;">
        <q-toolbar-title class="text-center">
          <q-img class="main-logo vertical-middle" alt="Logo Principal" :src="logo" fit="contain" />
        </q-toolbar-title>
      </q-toolbar>
    </q-card>

    <q-card flat bordered class="text-center q-pa-md q-ma-md">
      <div class="box">
        <div class="icon-container row justify-center">
          <q-img class="icon" src="resources/img/maintenance-announce.png"></q-img>
        </div>
        <h1 class="title">Em breve novidades!</h1>
        <div class="text-grey-9">
          <p class="sub-title q-px-sm">Estamos fazendo algumas melhorias para oferecer uma experiência ainda melhor.
            Voltamos em breve!</p>
          <div v-if="Object.keys(socials).length > 0">
            <p class="caption">Enquanto isso, acompanhe-nos nas redes sociais:</p>
            <div class="row flex-center">
              <div v-for="(social, idx) in socials" :key="idx" class="q-pa-sm">
                <q-btn outline round :icon="`fab fa-${idx}`" size='12px' :href="social" target='_blank' />
              </div>
            </div>
          </div>
        </div>
      </div>
    </q-card>
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
      logo: null,
      socials: {},
    }
  },

  methods: {
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

    getLogo() {
      return this.$http.get(`${ENDPOINTS.SETTINGS.SINGLE}/institution/logo`)
        .then((response) => {
          this.logo = response.data.tx_fieldvalue || 'resources/img/logo-horizontal.png';
        })
    },

    async getSocials() {
      // Emitting the loading event
      this.$emit('load', 'socials-read');
      try {
        const response = await this.$http.get(`${ENDPOINTS.SETTINGS.CONTEXT_OBJ}/socials`);
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
  },

  async mounted() {
    let isInMaintenance = await this.isInMaintenance();
    if (!isInMaintenance) {
      this.$router.push('/'); // Redirect to home if not in maintenance
      return;
    }
    await this.getLogo();
    await this.getSocials();
  },
}
</script>
<style>
.main-logo {
  height: 81px;
  max-width: 144px;
}

.box {
  padding: 40px 0;
}

.icon {

  max-width: 180px;
}

.title {
  margin-top: 3dvh;
  line-height: normal;
  font-size: 1.9em;
  font-weight: bold;
  text-align: center;
}

.caption {
  margin-bottom: 8px;
  font-size: 0.9em;
  line-height: normal;
}
</style>