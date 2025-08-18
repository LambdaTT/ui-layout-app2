<template>
  <q-page class="flex q-pa-sm text-center">
    <div animated>
      <q-card class="my-card" flat bordered>
        <q-card-section v-if="!hidePageHeader">
          <PageHeader :pageTitle="PageTitle" :breadcrumb="Breadcrumb" />
        </q-card-section>

        <div class="text-justify q-px-md q-pb-md">
          <slot></slot>
        </div>

        <!--Secondary Sections-->
        <div v-for="(section, name) in $slots" :key="name">
          <q-separator v-if="name != 'actions' && name != 'default'"></q-separator>
          <q-card-section v-if="name != 'actions' && name != 'default'">
            <slot :name="name"></slot>
          </q-card-section>
        </div>
      </q-card>
    </div>
  </q-page>
</template>

<script>
export default {
  name: 'ui-layoutapp2-page',

  props: {
    HidePageHeader: Boolean,
    PageTitle: String,
    Breadcrumb: Array
  },

  data() {
    return {
      hidePageHeader: true
    }
  },

  mounted() {
    this.hidePageHeader = this.HidePageHeader;
  }
}
</script>

<style scoped>
.q-page {
  display: block;
}
</style>