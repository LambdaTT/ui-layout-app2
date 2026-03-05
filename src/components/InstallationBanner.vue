<template>
  <div>
    <q-banner
      v-if="showInstallBanner"
      class="install-banner q-mb-md rounded-borders q-mx-sm q-mt-sm shadow-1"
      dense
    >
      <template v-slot:avatar>
        <q-icon name="fas fa-mobile-alt" color="primary" class="q-mt-xs" />
      </template>

      <div class="text-caption text-grey-9" style="line-height: 1.2">
        Instale este aplicativo em seu dispositivo para uma melhor experiência.
      </div>

      <template v-slot:action>
        <q-btn
          flat
          dense
          color="grey-6"
          icon="close"
          class="q-mr-xs"
          @click="dismissBanner"
          aria-label="Minimizar"
        />
        <q-btn
          unelevated
          size="sm"
          color="primary"
          icon="fas fa-download"
          label="Instalar"
          @click="openVideoModal"
          class="text-weight-bold rounded-borders q-px-sm"
        />
      </template>
    </q-banner>

    <q-dialog v-if="detectPlatform() == 'android'" v-model="videoModalOpen">
      <q-card>
        <q-toolbar>
          <q-toolbar-title>Como Instalar o App</q-toolbar-title>
          <q-btn flat round dense icon="close" v-close-popup />
        </q-toolbar>
        <q-card-section>
          <div class="text-subtitle1">
            <b>Passo 1:</b> Abra o menu do navegador clicando no ícone
            localizado no canto superior direito da tela
          </div>
          <div>
            <q-img src="~assets/installation/android1.png"></q-img>
          </div>
        </q-card-section>

        <q-separator></q-separator>

        <q-card-section>
          <div class="text-subtitle1">
            <b>Passo 2:</b> No menu suspenso que vai se abrir, localize a opção
            "Adicionar à Tela Inicial".
          </div>
          <div>
            <q-img src="~assets/installation/android2.png"></q-img>
          </div>
          <div class="q-mt-md">
            <b>Pronto!</b>
          </div>
        </q-card-section>

        <q-separator></q-separator>

        <q-card-actions align="right">
          <q-btn flat label="Fechar" @click="closeVideoModal" />
        </q-card-actions>
      </q-card>
    </q-dialog>

    <q-dialog v-if="detectPlatform() == 'ios'" v-model="videoModalOpen">
      <q-card>
        <q-toolbar>
          <q-toolbar-title>Como Instalar o App</q-toolbar-title>
          <q-btn flat round dense icon="close" v-close-popup />
        </q-toolbar>
        <q-card-section>
          <div class="text-subtitle1">
            <b>Passo 1:</b> Abra o menu de compartilhamento clicando no ícone
            localizado na parte de baixo da tela.
          </div>
          <div>
            <q-img src="~assets/installation/iphone1.png"></q-img>
          </div>
        </q-card-section>

        <q-separator></q-separator>

        <q-card-section>
          <div class="text-subtitle1">
            <b>Passo 2:</b> No menu que vai se abrir, localize a opção
            "Adicionar à Tela de Início".
          </div>
          <div>
            <q-img src="~assets/installation/iphone2.png"></q-img>
          </div>
          <div>
            <b>Pronto!</b>
          </div>
        </q-card-section>

        <q-separator></q-separator>

        <q-card-actions align="right">
          <q-btn flat label="Fechar" @click="closeVideoModal" />
        </q-card-actions>
      </q-card>
    </q-dialog>
  </div>
</template>

<script>
export default {
  name: "ui-layoutapp2-installationbanner",

  data() {
    return {
      showInstallBanner: false,
      videoModalOpen: false,
    };
  },

  methods: {
    isPwaInstalled() {
      return window.matchMedia("(display-mode: standalone)").matches;
    },

    detectPlatform() {
      const userAgent = navigator.userAgent || navigator.vendor || window.opera;
      if (/android/i.test(userAgent)) {
        return "android";
      } else if (/iPad|iPhone|iPod/.test(userAgent) && !window.MSStream) {
        return "ios";
      } else {
        return "other";
      }
    },

    dismissBanner() {
      this.showInstallBanner = false;
    },

    openVideoModal() {
      this.videoModalOpen = true;
    },

    closeVideoModal() {
      this.videoModalOpen = false;
    },
  },
  mounted() {
    if (!this.isPwaInstalled()) {
      this.showInstallBanner = true;
    }
  },
};
</script>

<style scoped>
.install-banner {
  background-color: #f8f9fa;
  border-left: 4px solid var(--q-primary);
}
</style>
