<template>
  <div>
    <xstate-add-to-cart />
    <add-to-cart />
  </div>
</template>

<script>
import xstateAddToCart from "./components/xstateAddToCart";
import addToCart from "./components/addToCart";

export default {
  name: "App",
  components: {
    addToCart,
    xstateAddToCart,
  },
};
</script>

<style></style>
