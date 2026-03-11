<template>
  <q-header elevated class="menu" :style="HeaderStyle">
    <q-toolbar :class="HeaderStyle && HeaderStyle.color ? '' : 'text-grey-9'" :style="HeaderStyle">
      <!-- Main logo -->
      <div class="menu-icon text-left">
        <q-btn class="q-pa-md menu" flat round icon="fas fa-bars" size="md" @click="$emit('toggle-drawer')">
        </q-btn>
      </div>

      <q-toolbar-title class="text-center">
        <q-btn dense flat class="full-width" @click="() => this.$router.push('/')">
          <q-img class="main-logo vertical-middle" alt="Logo Principal" :src="MainLogoSrc" :ratio="16 / 9"
            fit="contain" />
        </q-btn>
      </q-toolbar-title>

      <div class="menu-icon text-right">
        <!-- Btn: Header Options -->
        <slot name="header-options"></slot>
        <q-btn class="q-pa-md menu" flat round :icon="BtnActionsIcon" size="md">
          <q-menu>
            <q-list class="q-pa-sm text-grey-8 text-no-wrap">
              <q-item v-for="(a, idx) in Actions" :key="idx" v-close-popup clickable @click="a.fn()">
                <q-item-section side>
                  <q-icon v-if="a.icon" size="sm" :name="a.icon" />
                </q-item-section>
                <q-item-section v-if="a.title">{{ a.title }}</q-item-section>
              </q-item>
            </q-list>
          </q-menu>
        </q-btn>
      </div>
    </q-toolbar>
  </q-header>
</template>
<script>

export default {
  name: 'ui-layoutapp2-header',

  props: {
    BtnActionsIcon: String,
    Actions: Array,
    MainLogoSrc: String,
    HeaderStyle: {
      type: [Object, String],
      default: () => ({}),
    },
  },

}
</script>

<style scoped>
.main-logo {
  max-width: 150px;
}

.menu-icon {
  width: 100px;
}

.hover-notification:hover {
  background-color: #f5f5f5;
}

.q-item-section {
  white-space: normal;
}

.text-body1,
.text-caption {
  overflow-wrap: break-word;
  word-wrap: break-word;
  word-break: break-word;
}
</style>
