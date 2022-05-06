<script setup>
import PlusIcon from "./PlusIcon";
import MinusIcon from "./MinusIcon";
import { assign, createMachine } from "xstate";
import { useMachine } from "@xstate/vue";

const props = defineProps({
  productId: Number,
  qty: {
    type: Number,
    default: 1,
  },
});

const addToCartMachine = createMachine(
  {
    id: "addToCartMachine",
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
            target: "recentlyAdded",
          },
          onError: {
            actions: "assignErrors",
            target: "idle",
          },
        },
      },
      recentlyAdded: {
        after: {
          1000: {
            target: "idle",
          },
        },
        on: {
          UPDATE_QTY: {
            actions: "assignQty",
            cond: "minQty",
            target: "recentlyAdded",
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
        errors: (
          ctx,
          {
            data: {
              response: {
                data: { errors = {} },
                status = 500,
              },
            },
          }
        ) => {
          let result = {};

          if (status === 422) {
            for (let key in errors) {
              result[key] = errors[key][0];
            }
          } else {
            result.unknown = "Something went wrong. Please try again later.";
          }

          return result;
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
          class="p-1 rounded border-2 border-black hover:bg-yellow-400 focus:outline-none focus-visible:border-yellow-400"
          type="button"
          @click="send({ type: 'UPDATE_QTY', value: state.context.qty - 1 })"
        >
          <MinusIcon class="h-5 w-5" />
        </button>

        <input
          :value="state.context.qty"
          class="p-1 w-10 h-10 font-semibold text-center rounded border-2 border-black outline-none focus:ring-transparent focus:border-yellow-400"
          type="number"
          @change="
            (ev) =>
              send({ type: 'UPDATE_QTY', value: parseInt(ev.target.value, 10) })
          "
        />

        <button
          class="p-1 rounded border-2 border-black hover:bg-yellow-400 focus:outline-none focus-visible:border-yellow-400"
          type="button"
          @click="send({ type: 'UPDATE_QTY', value: state.context.qty + 1 })"
        >
          <PlusIcon class="w-5 h-5" />
        </button>
      </div>

      <button
        class="inline-flex justify-center items-center px-6 py-2.5 text-sm font-semibold text-black bg-yellow-400 rounded-full border-2 border-transparent hover:bg-black hover:text-white focus:outline-none focus-visible:border-black"
      >
        {{
          state.matches("recentlyAdded")
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
