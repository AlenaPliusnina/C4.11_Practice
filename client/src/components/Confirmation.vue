<template>
  <div>
    <b-alert variant="info" show dismissible @dismissed='showalert'>{{ message }}</b-alert>
  </div>
</template>

<script>
export default {
  props: ['message'],
  methods: {
    showalert() {
      this.$emit('showAlert');
    },
  },
};
</script>
