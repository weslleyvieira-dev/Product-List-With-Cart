<script setup>
import { ref, reactive, watch } from "vue";
import OrderConfirmationModal from "./components/OrderConfirmationModal.vue";
import dessertsData from "./data.json";

const desserts = ref(dessertsData);

const cart = reactive({
  items: [],
  totalItems: 0,
  totalOrder: 0.0,
});

const orderConfirmation = ref(false);

function debounce(func, delay) {
  let timeout;
  return function (...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, args), delay);
  };
}

const debouncedAddToCart = debounce((dessert) => addToCart(dessert), 100);
const debouncedSubFromCart = debounce((dessert) => subFromCart(dessert), 100);

function isInCart(dessert) {
  return cart.items.some((item) => item.name === dessert.name);
}

function addToCart(dessert) {
  const existingItem = cart.items.find((item) => item.name === dessert.name);
  if (existingItem) {
    existingItem.quantity++;
  } else {
    cart.items.push({ ...dessert, quantity: 1 });
  }
  cart.totalItems++;
  cart.totalOrder += dessert.price;
}

function subFromCart(dessert) {
  const existingItem = cart.items.find((item) => item.name === dessert.name);
  if (existingItem) {
    existingItem.quantity--;
    cart.totalItems--;
    cart.totalOrder -= dessert.price;
    if (existingItem.quantity <= 0) {
      cart.items = cart.items.filter((item) => item.name !== dessert.name);
    }
  }
}

function removeFromCart(dessert) {
  const existingItem = cart.items.find((item) => item.name === dessert.name);
  if (existingItem) {
    cart.totalItems -= existingItem.quantity;
    cart.totalOrder -= existingItem.price * existingItem.quantity;
    cart.items = cart.items.filter((item) => item.name !== dessert.name);
  }
}

function confirmOrder() {
  window.scrollTo({ top: 0, behavior: "smooth" });
  orderConfirmation.value = true;
}

function resetCart() {
  cart.items = [];
  cart.totalItems = 0;
  cart.totalOrder = 0.0;
  window.location.reload();
}

watch(orderConfirmation, (isOpen) => {
  if (isOpen) {
    setTimeout(() => {
      document.body.classList.add("no-scroll");
    }, 1000);
  } else {
    document.body.classList.remove("no-scroll");
  }
});
</script>

<template>
  <main>
    <div class="desserts">
      <h1>Desserts</h1>
      <div class="desserts-items">
        <div
          class="dessert-item"
          v-for="(dessert, index) in desserts"
          :key="index"
        >
          <div class="dessert-image-button">
            <picture class="dessert-image">
              <source
                media="(min-width: 1024px)"
                :srcset="dessert.image.desktop"
              />
              <source
                media="(max-width: 768px)"
                :srcset="dessert.image.mobile"
              />
              <img
                :src="dessert.image.mobile"
                :alt="dessert.name"
                class="dessert-image"
                :class="{ selected: isInCart(dessert) }"
              />
            </picture>
            <button
              v-if="!isInCart(dessert)"
              v-on:click="debouncedAddToCart(dessert)"
              class="dessert-button"
            >
              <img
                src="./assets/images/icon-add-to-cart.svg"
                alt="Add to Cart"
              /><strong>Add to Cart</strong>
            </button>
            <div class="dessert-control" v-else-if="isInCart(dessert)">
              <button
                class="dessert-control-decrement"
                v-on:click="debouncedSubFromCart(dessert)"
              >
                <img src="./assets/images/icon-decrement-quantity.svg" />
              </button>
              {{
                cart.items.find((item) => item.name === dessert.name).quantity
              }}
              <button
                class="dessert-control-increment"
                v-on:click="debouncedAddToCart(dessert)"
              >
                <img src="./assets/images/icon-increment-quantity.svg" />
              </button>
            </div>
          </div>
          <p class="dessert-category">{{ dessert.category }}</p>
          <h2 class="dessert-name">{{ dessert.name }}</h2>
          <p class="dessert-price">${{ dessert.price.toFixed(2) }}</p>
        </div>
      </div>
    </div>
    <div class="cart">
      <h1>Your Cart ({{ cart.totalItems }})</h1>
      <div v-if="!cart.totalItems" class="empty-cart">
        <img src="./assets/images/illustration-empty-cart.svg" />
        <p>Your added items will appear here</p>
      </div>
      <div v-if="cart.totalItems" class="cart-items">
        <div class="cart-item" v-for="(item, index) in cart.items" :key="index">
          <div class="cart-item-details">
            <h2 class="cart-item-name">{{ item.name }}</h2>
            <div class="cart-item-price">
              <p class="cart-item-quantity">{{ item.quantity }}x</p>
              <p>@ ${{ item.price.toFixed(2) }}</p>
              <p class="cart-item-total">
                ${{ (item.quantity * item.price).toFixed(2) }}
              </p>
            </div>
          </div>
          <button class="cart-item-remove" @click="removeFromCart(item)">
            <img src="./assets/images/icon-remove-item.svg" alt="Remove Item" />
          </button>
        </div>
      </div>
      <div v-if="cart.totalItems" class="cart-total-order">
        <p>Order Total</p>
        <h2 class="cart-total-order-value">
          ${{ cart.totalOrder.toFixed(2) }}
        </h2>
      </div>
      <div v-if="cart.totalItems" class="carbon-neutral">
        <img src="./assets/images/icon-carbon-neutral.svg" />
        <p>This is a <strong>carbon-neutral</strong> delivery</p>
      </div>
      <button
        v-if="cart.totalItems"
        class="confirm-order"
        v-on:click="confirmOrder"
      >
        Confirm Order
      </button>
    </div>
    <div v-if="orderConfirmation">
      <OrderConfirmationModal
        :cart="cart"
        @resetCart="resetCart"
        @close="orderConfirmation = false"
      />
    </div>
  </main>
</template>

<style scoped>
main {
  display: flex;
  flex-direction: row;
  background-color: hsl(13, 31%, 94%);
  gap: 1rem;
  padding: 2rem 4rem;
}

/* ----- DESSERTS ----- */
.desserts {
  display: flex;
  flex-direction: column;
  padding: 2rem;
  width: 100%;
}

.desserts h1 {
  font-size: 2rem;
  margin-bottom: 2rem;
}

.desserts-items {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 1rem 2rem;
}

.dessert-item {
  resize: none;
  box-sizing: border-box;
}

.dessert-image {
  width: 15rem;
  border-radius: 1rem;
}

.dessert-image.selected {
  border: 3px solid hsl(14, 86%, 42%);
  resize: none;
  box-sizing: border-box;
}

.dessert-image-button {
  position: relative;
}

.dessert-button {
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 2rem;
  border: 1px solid hsl(14, 86%, 42%);
  width: 10rem;
  height: 3rem;
  cursor: pointer;
  padding: 0;
  z-index: 2;
  position: absolute;
  bottom: -1rem;
  left: 50%;
  transform: translateX(-50%);
  gap: 0.5rem;
  background-color: hsl(20, 50%, 98%);
}

.dessert-button:hover {
  background-color: hsl(20, 50%, 98%);
  color: hsl(14, 86%, 42%);
}

.dessert-button strong {
  font-weight: 600;
}

.dessert-control {
  display: flex;
  align-items: center;
  justify-content: space-between;
  border-radius: 2rem;
  border: 1px solid hsl(14, 86%, 42%);
  width: 10rem;
  height: 3rem;
  padding: 0 1rem;
  resize: none;
  box-sizing: border-box;
  z-index: 2;
  position: absolute;
  bottom: -1rem;
  left: 50%;
  transform: translateX(-50%);
  background-color: hsl(14, 86%, 42%);
  color: hsl(20, 50%, 98%);
}

.dessert-control-decrement,
.dessert-control-increment {
  display: flex;
  align-items: center;
  justify-content: center;
  background-color: transparent;
  width: 1.3rem;
  height: 1.3rem;
  border: 1px solid hsl(20, 50%, 98%);
  border-radius: 2rem;
}

.dessert-control-decrement img,
.dessert-control-increment img {
  width: 0.7rem;
  height: 0.7rem;
}

.dessert-control-decrement:hover,
.dessert-control-increment:hover {
  background-color: hsl(20, 50%, 98%);
  cursor: pointer;
}

.dessert-control-decrement:hover img,
.dessert-control-increment:hover img {
  filter: brightness(0) saturate(100%) invert(29%) sepia(97%) saturate(6026%)
    hue-rotate(19deg) brightness(95%) contrast(88%);
}

.dessert-category {
  font-weight: lighter;
  margin: 2rem 0 0;
}

.dessert-name {
  font-weight: 700;
  margin: 0.5rem 0;
}

.dessert-price {
  font-weight: 700;
  color: hsl(14, 86%, 42%);
  margin-top: 0;
}

/* ----- CART ----- */
.cart {
  display: flex;
  flex-direction: column;
  padding: 0.5rem 2rem;
  background-color: hsl(20, 50%, 98%);
  border-radius: 1rem;
  width: 100%;
  height: max-content;
  margin: 3rem 0;
  margin-right: 2rem;
}

.cart h1 {
  font-size: 1.8rem;
  margin-bottom: 1rem;
  color: hsl(14, 86%, 42%);
}

.empty-cart {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 2rem 0;
  gap: 1rem;
}

.empty-cart img {
  width: 10rem;
  height: 10rem;
}

.empty-cart p {
  font-weight: 600;
  color: hsl(12, 20%, 44%);
}

.cart-items {
  display: flex;
  flex-direction: column;
}

.cart-item {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  border-bottom: 1px solid hsl(13, 31%, 90%);
  padding: 0.5rem 0;
}

.cart-item-details {
  display: flex;
  flex-direction: column;
}

.cart-item-name {
  font-weight: 600;
  margin-bottom: 0;
}

.cart-item-price {
  display: flex;
  flex-direction: row;
  gap: 1rem;
  color: hsl(7, 20%, 60%);
}

.cart-item-price * {
  margin: 0.7rem 0 1rem;
}

.cart-item-quantity {
  font-weight: 700;
  color: hsl(14, 86%, 42%);
}

.cart-item-total {
  font-weight: 600;
  color: hsl(12, 20%, 44%);
}

.cart-item-remove {
  display: flex;
  justify-content: center;
  align-items: center;
  width: 1.3rem;
  height: 1.3rem;
  background-color: hsl(20, 50%, 98%);
  border-radius: 3rem;
  border: 1px solid hsl(12, 20%, 44%);
}

.cart-item-remove img {
  width: 0.7rem;
  height: 0.7rem;
}

.cart-item-remove:hover {
  border-color: hsl(14, 65%, 9%);
  cursor: pointer;
}

.cart-item-remove:hover img {
  filter: brightness(0) saturate(100%) invert(7%) sepia(32%) saturate(1424%)
    hue-rotate(333deg) brightness(94%) contrast(99%);
}

.cart-total-order {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
}

.cart-total-order * {
  margin: 1.7rem 0;
}

.cart-total-order h2 {
  font-size: 1.7rem;
}

.carbon-neutral {
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: center;
  gap: 0.7rem;
  background-color: hsl(13, 31%, 94%);
  border-radius: 0.5rem;
  margin: 0 0 1.7rem;
  padding: 0.3rem 0;
}

.carbon-neutral img {
  width: 1.3rem;
  height: 1.3rem;
}

.carbon-neutral p strong {
  font-weight: 600;
}

.confirm-order {
  background-color: hsl(14, 86%, 42%);
  color: hsl(20, 50%, 98%);
  padding: 1rem 0;
  margin: 0 0 1rem;
  border-style: solid;
  border-radius: 2rem;
  font-weight: 600;
}

.confirm-order:hover {
  background-color: hsl(14, 86%, 32%);
  color: hsl(14, 25%, 82%);
  cursor: pointer;
}

@media (max-width: 1024px) {
  main {
    flex-direction: column;
    align-items: center;
    gap: 1rem;
    padding: 0;
    resize: none;
    box-sizing: border-box;
    overflow-x: hidden;
  }

  /* ----- DESSERTS ----- */
  .desserts {
    padding: 0 2rem;
    width: 90%;
  }

  .desserts h1 {
    font-size: 2.5rem;
    margin: 2rem 0;
  }

  .desserts-items {
    display: flex;
    flex-direction: column;
    gap: 0.5rem;
  }

  .dessert-image {
    width: 100%;
  }

  .dessert-category {
    margin: 1.5rem 0 0;
  }

  /* ----- CARTS ----- */
  .cart {
    padding: 0 2rem;
    width: 90%;
    margin: 0 0 2rem;
    resize: none;
    box-sizing: border-box;
  }
}
</style>
