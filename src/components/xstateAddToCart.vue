<script setup>
import { assign, createMachine } from "xstate";
import { useMachine } from "@xstate/vue";
import axios from "axios";

const props = defineProps({
  productId: Number,
  qty: {
    type: Number,
    defauld: 1,
  },
});

const addToCartMachine = createMachine(
  {
    id: "addToCartMachine ",
    context: {
      productId: props.productId,
      qty: props.qty,
      errors: {},
    },

    initial: "idle",
    states: {
      idle: {
        on: {
          UPDATE_QTY: {
            actions: "assignQty",
            cond: "minQty",
            target: "idle",
          },
          ADD_TO_CART: {
            target: "adding",
          },
        },
      },
      adding: {
        entry: "clearErrors",
        invoke: {
          id: "addToCart",
          src: "addToCart",
          onDone: {
            target: "recentyAdded",
          },
          onError: {
            actions: "assignErrors",
            target: "idle",
          },
        },
      },
      recentyAdded: {
        after: {
          1000: {
            target: "idle",
          },
        },
        on: {
          UPDATE_QTY: {
            actions: "assignQty",
            cond: "minQty",
            target: "recentyAdded",
          },
        },
      },
    },
  },
  {
    actions: {
      assignQty: assign({
        qty: (ctx, event) => event.value,
      }),
      clearErrors: assign({
        errors: {},
      }),
      assignErrors: assign({
        errors: (ctx, event) => {
          console.log(event);
        },
      }),
    },
    services: {
      addToCart: (ctx) =>
        axios.post("/api/cart-items", {
          qty: ctx.qty,
          productId: ctx.productId,
        }),
    },
    guards: {
      minQty: (ctx, event) => event.value >= 1,
    },
  }
);
const { state, send, service } = useMachine(addToCartMachine);
</script>

<template>
  <form @submit.prevent="send('ADD_TO_CART')">
    <div class="flex space-x-4">
      <div class="flex items-center space-x-2">
        <button
          @click="send({ type: 'UPDATE_QTY', value: state.context.qty - 1 })"
          class="p-1 rounded border-2 border-black hover:bg-yellow-400 focus:outline-none focus-visible:border-yellow-400"
          type="button"
        >
          <img
            src="../assets/icons8-минус-48.png"
            class="h-5 w-5"
            alt="минус"
          />
        </button>

        <input
          @change="
            (ev) =>
              send({ type: 'UPDATE_QTY', value: parseInt(ev.target.value, 10) })
          "
          :value="state.context.qty"
          class="p-1 w-10 h-10 font-semibold text-center rounded border-2 border-black outline-none focus:ring-transparent focus:border-yellow-400"
          type="number"
        />

        <button
          @click="send({ type: 'UPDATE_QTY', value: state.context.qty + 1 })"
          class="p-1 rounded border-2 border-black hover:bg-yellow-400 focus:outline-none focus-visible:border-yellow-400"
          type="button"
        >
          <img src="../assets/icons8-plus-48.png" class="w-5 h-5" alt="плюс" />
        </button>
      </div>

      <button
        class="inline-flex justify-center items-center px-6 py-2.5 text-sm font-semibold text-black bg-yellow-400 rounded-full border-2 border-transparent hover:bg-black hover:text-white focus:outline-none focus-visible:border-black"
      >
        {{
          state.matches("recentyAdded")
            ? state.context.qty > 1
              ? "Products were added"
              : "Product was added"
            : "Add to cart"
        }}
      </button>
    </div>

    <div
      v-if="state.context.errors.productId"
      class="mt-2 text-sm text-red-600"
    >
      {{ state.context.errors.productId }}
    </div>
    <div v-if="state.context.errors.qty" class="mt-2 text-sm text-red-600">
      {{ state.context.errors.qty }}
    </div>
    <div v-if="state.context.errors.unknown" class="mt-2 text-sm text-red-600">
      {{ state.context.errors.unknown }}
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
