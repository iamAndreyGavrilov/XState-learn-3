<script setup>
import PlusIcon from "./PlusIcon";
import MinusIcon from "./MinusIcon";
import {ref} from "vue";
 
const props = defineProps({
  productId: Number,
  qty: {
    type: Number,
    default: 1,
  }
});
 
const qty = ref(props.qty);
const errors = ref({});
const isSubmitting = ref(false);
const isRecentlyAdded = ref(false);
 
async function onSubmit() {
  errors.value = {};
  if (isSubmitting.value || isRecentlyAdded.value) return;
 
  try {
    isSubmitting.value = true;
    await axios.post('/api/cart-items', {qty: qty.value, productId: props.productId});
    isRecentlyAdded.value = true;
    setTimeout(() => isRecentlyAdded.value = false, 1000);
  } catch ({response: {data: {errors: err = {}}, status = 500}}) {
    if (status === 422) {
      for (let key in err) {
        errors.value[key] = err[key][0];
      }
    } else {
      errors.value.unknown = "Something went wrong. Please try again later."
    }
  }
 
  isSubmitting.value = false;
}
</script>
 
<template>
  <form @submit.prevent="onSubmit()">
    <div class="flex space-x-4">
      <div class="flex items-center space-x-2">
        <button
          class="p-1 rounded border-2 border-black hover:bg-yellow-400 focus:outline-none focus-visible:border-yellow-400"
          type="button"
          @click="qty-1 > 0 ? qty-- :qty">
          <MinusIcon class="h-5 w-5"/>
        </button>
 
        <input
          v-model="qty"
          class="p-1 w-10 h-10 font-semibold text-center rounded border-2 border-black outline-none focus:ring-transparent focus:border-yellow-400"
          type="number">
 
        <button
          class="p-1 rounded border-2 border-black hover:bg-yellow-400 focus:outline-none focus-visible:border-yellow-400"
          type="button"
          @click="qty++">
          <PlusIcon class="w-5 h-5"/>
        </button>
      </div>
 
      <button
        class="inline-flex justify-center items-center px-6 py-2.5 text-sm font-semibold text-black bg-yellow-400 rounded-full border-2 border-transparent hover:bg-black hover:text-white focus:outline-none focus-visible:border-black">
        {{ isRecentlyAdded ? (qty > 1 ? 'Products were added' : 'Product was added') : 'Add to cart' }}
      </button>
    </div>
 
    <div v-if="errors.productId" class="mt-2 text-sm text-red-600">
      {{ errors.productId }}
    </div>
    <div v-if="errors.qty" class="mt-2 text-sm text-red-600">
      {{ errors.qty }}
    </div>
    <div v-if="errors.unknown" class="mt-2 text-sm text-red-600">
      {{ errors.unknown }}
    </div>
  </form>
</template>
 
<style scoped>
input[type="number"]::-webkit-inner-spin-button,
input[type="number"]::-webkit-outer-spin-button {
  -webkit-appearance: none;
  margin: 0;
}
 
input[type="number"] {
  -moz-appearance: textfield;
}
</style>