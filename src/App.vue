<template>
  <v-app>
    <v-main>
      <shopping-component />
    </v-main>
  </v-app>
</template>

<script lang="ts">
import { defineComponent } from "vue";
import ShoppingComponent from "./components/ShoppingComponent.vue";

export default defineComponent({
  name: "App",
  components: {
    ShoppingComponent,
  },

  data() {
    return {
      //
    };
  },
});
</script>
