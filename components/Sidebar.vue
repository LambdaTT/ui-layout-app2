<template>
  <q-drawer class="menu text-body2" v-model="drawerState" :breakpoint="1920"
    @update:model-value="updateModel">
      <div class="column no-wrap" style="height: 100%;">
        <!-- Logo -->
        <div class="col-auto text-center q-pa-lg" v-on:click="$router.push('/')">
          <q-img class="main-logo vertical-middle full-width" alt="Logo do sindicato" :src="MainLogoSrc" />
        </div>
        <div class="col-auto">
          <q-separator></q-separator>
        </div>
        <!-- Nav Items: -->
        <div class="col" style="overflow-y: auto;">
          <q-list>
            <div v-for="(item, index) in NavItems" :key="index">
              <q-expansion-item v-if="item.subitems && item.subitems.length > 0" :icon="item.icon" :label="item.title">
                <q-list padding>
                  <q-item v-for="(subitem, idx) in item.subitems" :key="idx" clickable @click="subitem.fn">
                    <q-item-section avatar>
                      &nbsp;
                    </q-item-section>
                    <q-item-section>
                      <span>
                        <q-icon name="fas fa-chevron-right" size="10px"></q-icon> 
                        {{ subitem.title }}
                      </span>
                    </q-item-section>
                  </q-item>
                </q-list>
              </q-expansion-item>
              <q-item v-else clickable @click="item.fn">
                <q-item-section avatar>
                  <q-icon v-if="item.icon" :name="item.icon"></q-icon>
                </q-item-section>
                <q-item-section>
                  <span>{{ item.title }}</span>
                </q-item-section>
              </q-item>
            </div>
          </q-list>
        </div>
        
        <!-- Footer -->
        <div class="col-auto socialsFooter">
          <div class="row justify-center">
            <div class="col-12 text-center q-pt-sm">
              <b>Nosso sindicato</b> nas redes sociais
            </div>
            <div v-for="(social, idx) in Socials" :key="idx" class="q-pa-xs">
              <q-btn outline round :icon="`fab fa-${idx}`" size='12px' :href="social" target='_blank' />
            </div>
          </div>
        </div>
      </div>
  </q-drawer>
</template>

<script>

export default {
  name: 'ui-layoutapp2-sidebar',
  data() {
    return {
      drawerState: false,
    }
  },

  props: {
    MainLogoSrc: String,
    NavItems: Array,
    Socials: Object,
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
    
    openLink(url) {
      window.open(url, '_blank');
    }
  },

}
</script>