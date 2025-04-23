<script setup>
defineProps({
  cart: {
    type: Object,
    required: true,
  },
});

const emit = defineEmits(["close"]);

function closeModal() {
  emit("close");
  emit("resetCart");
}
</script>

<template>
  <div class="modal-overlay">
    <div class="modal">
      <div class="modal-header">
        <img
          class="modal-icon"
          src="../assets/images/icon-order-confirmed.svg"
        />
        <h1 class="modal-title">Order Confirmed</h1>
        <p class="modal-message">We hope you enjoy your food!</p>
      </div>
      <div class="order-itens">
        <div
          class="order-item"
          v-for="(item, index) in cart.items"
          :key="index"
        >
          <div class="order-item-view">
            <img
              :src="item.image.thumbnail"
              :alt="item.name"
              class="item-image"
            />
            <div class="order-item-details">
              <h2 class="order-item-name">{{ item.name }}</h2>
              <div class="order-item-price">
                <p class="order-item-quantity">{{ item.quantity }}x</p>
                <p class="order-item-value">@ ${{ item.price.toFixed(2) }}</p>
              </div>
            </div>
          </div>
          <p class="order-item-total">
            ${{ (item.quantity * item.price).toFixed(2) }}
          </p>
        </div>
      </div>
      <div class="order-total">
        <p>Order Total</p>
        <h2>${{ cart.totalOrder.toFixed(2) }}</h2>
      </div>
      <button class="new-order" v-on:click="closeModal">Start New Order</button>
    </div>
  </div>
</template>

<style scoped>
.modal-overlay {
  display: flex;
  justify-content: center;
  align-items: center;
  position: fixed;
  top: 0;
  left: 0;
  width: 100dvw;
  height: 100dvh;
  background-color: hsl(14, 65%, 9%, 0.4);
  z-index: 10;
}

.modal {
  display: flex;
  flex-direction: column;
  justify-content: center;
  background-color: hsl(20, 50%, 98%);
  padding: 2rem 2.5rem;
  border-radius: 1rem;
  width: min(35rem, 70dvw);
  max-height: 72dvh;
}

.modal-icon {
  width: 3rem;
}

.modal-title {
  font-size: 2rem;
  margin: 1rem 0 0;
}

.modal-message {
  font-weight: lighter;
  margin: 0.5rem 0;
}

.order-itens {
  display: flex;
  flex-direction: column;
  background-color: hsl(13, 31%, 94%);
  padding: 1rem 1.5rem 0;
  border-radius: 1rem 1rem 0 0;
  margin: 2rem 0 0;
  flex-grow: 1;
  overflow-y: scroll;
}

.order-item {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: space-between;
  padding: 1rem 0;
  border-bottom: 1px solid hsl(13, 31%, 90%);
}

.order-item-view {
  display: flex;
  flex-direction: row;
  align-items: center;
  gap: 1.5rem;
}

.item-image {
  height: 3.5rem;
  border-radius: 0.5rem;
}

.order-item-details {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.order-item-name {
  font-weight: 600;
  margin: 0;
}

.order-item-price {
  display: flex;
  flex-direction: row;
  gap: 1.5rem;
}

.order-item-price * {
  margin: 0;
}

.order-item-quantity {
  font-weight: 600;
  color: hsl(14, 86%, 42%);
}

.order-item-value {
  font-weight: 400;
  color: hsl(12, 20%, 44%);
}

.order-item-total {
  font-weight: 600;
}

.order-total {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  background-color: hsl(13, 31%, 94%);
  border-radius: 0 0 1rem 1rem;
  padding: 1.5rem 1.5rem;
  margin: 0 0 2rem;
}

.order-total p {
  margin: 0;
}

.order-total h2 {
  font-size: 1.5rem;
  margin: 0;
}

.new-order {
  background-color: hsl(14, 86%, 42%);
  color: hsl(20, 50%, 98%);
  padding: 1rem 0;
  margin: 0;
  border-style: solid;
  border-radius: 2rem;
  font-weight: 600;
}

.new-order:hover {
  background-color: hsl(14, 86%, 32%);
  color: hsl(14, 25%, 82%);
  cursor: pointer;
}

@media (max-width: 1024px) {
  .modal-overlay {
    align-items: flex-end;
  }

  .modal {
    width: 100%;
    height: max-content;
    max-height: 80dvh;
    overflow-y: visible;
    padding: 1.5rem;
  }

  .order-itens {
    overflow-x: hidden;
  }

  .order-item-details {
    flex: 1 1 0;
    min-width: 0;
  }

  .order-item-view {
    display: flex;
    flex: 1 1 0;
    min-width: 0;
    gap: 1rem;
  }

  .order-item-name {
    display: block;
    min-width: 0;
    width: 85%;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }

  .order-item-total {
    flex-shrink: 0;
  }
}
</style>
