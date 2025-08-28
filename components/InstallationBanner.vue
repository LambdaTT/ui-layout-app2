<template>
  <div>
    <q-banner v-if="showInstallBanner" class="q-mb-md" dense>
      <div class="text-justify">
        <q-icon name="fas fa-info-circle" size="xs" class="q-ma-xs" />
        Instale este aplicativo em seu dispositivo para uma melhor experiência.
      </div>
      <q-btn class="q-ma-sm" color="primary" icon="fas fa-download" @click="openVideoModal" label="Instalar" />
    </q-banner>

    <q-dialog v-if="detectPlatform() == 'android'" v-model="videoModalOpen">
      <q-card>
        <q-toolbar>
          <q-toolbar-title>Como Instalar o App</q-toolbar-title>
          <q-btn flat round dense icon="close" v-close-popup />
        </q-toolbar>
        <q-card-section>
          <div class="text-subtitle1">
            <b>Passo 1:</b> Abra o menu do navegador clicando no ícone localizado no canto superior direito da tela
          </div>
          <div>
            <q-img src="~assets/installation/android1.png"></q-img>
          </div>
        </q-card-section>

        <q-separator></q-separator>

        <q-card-section>
          <div class="text-subtitle1">
            <b>Passo 2:</b> No menu suspenso que vai se abrir, localize a opção "Adicionar à Tela Inicial".
          </div>
          <div>
            <q-img src="~assets/installation/android2.png"></q-img>
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
            <b>Passo 1:</b> Abra o menu de compartilhamento clicando no ícone localizado na parte de baixo da tela.
          </div>
          <div>
            <q-img src="~assets/installation/iphone1.png"></q-img>
          </div>
        </q-card-section>

        <q-separator></q-separator>

        <q-card-section>
          <div class="text-subtitle1">
            <b>Passo 2:</b> No menu que vai se abrir, localize a opção "Adicionar à Tela de Início".
          </div>
          <div>
            <q-img src="~assets/installation/iphone2.png"></q-img>
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
  name: 'ui-layoutapp2-installationbanner',

  data() {
    return {
      showInstallBanner: false,
      videoModalOpen: false
    };
  },

  methods: {
    isPwaInstalled() {
      return window.matchMedia('(display-mode: standalone)').matches;
    },

    detectPlatform() {
      const userAgent = navigator.userAgent || navigator.vendor || window.opera;
      if (/android/i.test(userAgent)) {
        return 'android';
      } else if (/iPad|iPhone|iPod/.test(userAgent) && !window.MSStream) {
        return 'ios';
      } else {
        return 'other';
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
    }
  },
  mounted() {
    if (!this.isPwaInstalled()) {
      this.showInstallBanner = true;
    }
  }
};
</script>

<style scoped>
.q-banner {
  background-color: #343a40;
  color: #f8f9fa;
}
</style>
