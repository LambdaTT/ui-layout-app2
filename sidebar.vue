<template>
  <q-drawer class="bg-grey-3 text-grey-10 text-body2" v-model="drawerState" :breakpoint="1920" bordered
    @update:model-value="updateModel">
    <q-scroll-area class="fit" :horizontal-thumb-style="{ opacity: 0 }">
      <div class="text-center q-pa-lg">
        <img class="main-logo vertical-middle full-width" alt="Logo do sindicato"
          src="~assets/demo/logo-sindicato.png" />
      </div>
      <q-separator></q-separator>
      <q-list>
        <!-- Link: Home -->
        <q-item clickable @click="navTo('/')">
          <q-item-section avatar>
            <q-icon name="fas fa-home"></q-icon>
          </q-item-section>
          <q-item-section>
            <span>Página Inicial</span>
          </q-item-section>
        </q-item>

        <q-expansion-item icon="fas fa-building-columns" label="O Sindicato">
          <q-list padding>
            <q-item clickable @click="navTo('/aboutus')">
              <q-item-section avatar>
                &nbsp;
              </q-item-section>
              <q-item-section>
                <span><q-icon name="fas fa-chevron-right" size="10px"></q-icon> Sobre Nós</span>
              </q-item-section>
            </q-item>
            <q-item clickable @click="navTo('/directorship')">
              <q-item-section avatar>
                &nbsp;
              </q-item-section>
              <q-item-section>
                <span><q-icon name="fas fa-chevron-right" size="10px"></q-icon> Conheça a Diretoria</span>
              </q-item-section>
            </q-item>

          </q-list>
        </q-expansion-item>

        <q-item clickable @click="navTo('/events')">
          <q-item-section avatar>
            <q-icon name="fas fa-calendar-day"></q-icon>
          </q-item-section>
          <q-item-section>
            <span>Calendário de Eventos</span>
          </q-item-section>
        </q-item>
        <q-item clickable @click="navTo('/benefits')">
          <q-item-section avatar>
            <q-icon name="fas fa-handshake"></q-icon>
          </q-item-section>
          <q-item-section>
            <span>Convênios e Parcerias</span>
          </q-item-section>
        </q-item>
        <q-item clickable @click="navTo('/associateme')">
          <q-item-section avatar>
            <q-icon name="fas fa-user-plus"></q-icon>
          </q-item-section>
          <q-item-section>
            <span>Quero me Associar</span>
          </q-item-section>
        </q-item>
        <q-item clickable @click="navTo('/links')">
          <q-item-section avatar>
            <q-icon name="fas fa-star"></q-icon>
          </q-item-section>
          <q-item-section>
            <span>Links Importantes</span>
          </q-item-section>
        </q-item>
        <q-item clickable @click="navTo('/contact')">
          <q-item-section avatar>
            <q-icon name="fas fa-headset"></q-icon>
          </q-item-section>
          <q-item-section>
            <span>Fale Conosco</span>
          </q-item-section>
        </q-item>

      </q-list>
      <!-- <q-footer class="q-pa-xs text-center">
        <q-separator></q-separator>
        <div class="row">
          <div class="col-6">
            <small>SIGS App</small>
          </div>
          <div class="col-6">
            <small>Desenvolvido por: Lambda TT</small>
          </div>
        </div>
      </q-footer> -->
    </q-scroll-area>
  </q-drawer>
</template>

<script>

export default {
  name: 'navbar-component',
  data() {
    return {
      drawerState: false,
    }
  },

  props: {
    modelValue: Boolean,
  },

  watch: {
    modelValue(value) {
      this.drawerState = value;
    }
  },

  methods: {
    updateModel(val) {
      this.$emit('update:model-value', val);
    },

    navTo(url) {
      this.$router.push(url);
    },
  },

  created() {
  },

}
</script>

<style scoped></style>
