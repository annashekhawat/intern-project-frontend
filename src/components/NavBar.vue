<template>
  <div>
    <b-navbar toggleable="lg" type="dark" variant="dark">
      <b-container>
        <b-navbar-brand href="#">Triton</b-navbar-brand>
        <b-navbar-toggle target="nav-collapse"></b-navbar-toggle>
      </b-container>
    </b-navbar>
  </div>
</template>
<script>
export default {
  data() {
    return {
      meal: "Search for merchants"
    };
  },
  methods: {
    getMeal() {
      return "Search for merchants";
    }
  }
};
</script>
