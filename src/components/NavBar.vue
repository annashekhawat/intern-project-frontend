<template>
  <div>
    <b-navbar toggleable="lg" class="navbar">
      <b-container>
        <b-navbar-brand href="#" class="navbar-brand">Service Discovery</b-navbar-brand>
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

<style lang="scss" scoped>
.navbar {
  background-color: rgb(26, 31, 113);
}

.navbar-brand {
  color: white;
}

.navbar-brand:hover {
  color: rgb(199, 175, 140);
}
</style>