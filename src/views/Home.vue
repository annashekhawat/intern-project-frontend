<template>
  <div>
    <nav-bar/>
    <mo-vies-tabl/>
  </div>
</template>

<script>
// @ is an alias to /src
import MoViesTabl from "@/components/MoviesTable.vue";
import NavBar from "@/components/NavBar.vue";

export default {
  name: "home",
  components: {
    MoViesTabl,
    NavBar
  }
};
</script>