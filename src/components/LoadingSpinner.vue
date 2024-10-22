<template>
  <v-dialog v-model="loading" hide-overlay persistent width="80">
    <v-card class="d-flex justify-center align-center" height="80">
      <v-progress-circular
        :size="40"
        :width="4"
        color="primary"
        indeterminate
      ></v-progress-circular>
    </v-card>
  </v-dialog>
</template>

<script>
export default {
  props: {
    loading: {
      type: Boolean,
      required: true,
    },
  },
};
</script>

<style scoped>
/* You can style the spinner further here */
</style>
