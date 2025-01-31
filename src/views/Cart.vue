<script setup>
import { ref, onMounted, computed } from 'vue';
import axiosInstance from '@/axiosInstance';
import { useCart } from '@/composable/cart';
import { useRouter } from 'vue-router';
import useAuth from '@/composable/useAuth';

const { showMessage, countdown, checkAccessToken } = useAuth();
const { removeCartItem, createOrderFromCart } = useCart();
const cartItem = ref([]);
const idClient = localStorage.getItem('idClient');
const accessToken = localStorage.getItem('accessToken');
const router = useRouter();

// GET CART ARTICLES
const fetchCartItems = async () => {
  try {
    if (!checkAccessToken(accessToken)) {
      return;
    }
    const response = await axiosInstance.get(`cart/client/${idClient}`, {
      headers: {
        'Authorization': `Bearer ${localStorage.getItem('accessToken')}`
      }
    });
    cartItem.value = response.data;
    console.log(cartItem.value);
  } catch (error) {
    console.error('Error fetching cart items:', error);
  }
};
//REMOVE ARTICLE FROM CART
const removeItem = async (idCart, idArticle) => {
  try {
    // if (!checkAccessToken(accessToken)) {
    //   return;
    // }
    await removeCartItem(idCart, idArticle);
    await fetchCartItems(); // Refresh cart after removal
  } catch (error) {
    console.error('Error removing item:', error);
  }
};
// CREATE ORDER
const createOrder = async (idCart) => {
  try {

    const idOrder = await createOrderFromCart(idCart);
    await fetchCartItems();
    router.push({ name: 'order', params: { id: idOrder } });
  } catch (error) {
    console.error('Error creating order');
  }
};

const isCartEmpty = computed(() => {
  return !cartItem.value || !cartItem.value.cartArticles || cartItem.value.cartArticles.length === 0;
});

const formatPrice = (price) => {
  return parseFloat(price).toFixed(2);
};

onMounted(fetchCartItems);
</script>

<template>
  <!-- ERROR MESSAGE -->
  <div v-if="showMessage" class="absolute z-50 inset-0 flex items-center justify-center backdrop-blur-lg">
    <div class="bg-card p-12 rounded-lg shadow-inner-strong text-white">
      <p class="text-md text-center font-bold text-warning">Session expired. Redirecting...</p>
      <p class="text-center text-xs text-tGray">Returning to homepage in {{ countdown }} seconds</p>
    </div>
  </div>

  <!-- CONTAINER -->
  <div class="text-white py-24 min-h-full flex flex-col justify-center items-center ">
    <div class="flex flex-col justify-center items-center">
      <!-- TITLE -->
      <h1 class="text-2xl font-bold mb-4">Your Cart</h1>

      <!-- NO CART FOR CLIENT -->
      <div v-if="isCartEmpty">
        <div class="bg-card p-20 rounded-lg flex flex-col items-center">
          <p class="text-tDarkGray text-sm">We are sorry but Your cart is empty </p>
          <button @click="router.push('/home')"
            class="mt-4 font-bold text-sm bg-primary text-white px-4 py-3 rounded-full hover:shadow-inner-strong">
            Back to the articles
          </button>
        </div>

      </div>

      <!-- ITEMS CART -->
      <div v-else>
        <div v-for="item in cartItem.cartArticles" :key="item.article.idArticle" class="p-4 mb-4">
          <!-- CARD -->
          <div class="flex flex-col items-center justify-between bg-card p-10 rounded-xl">
            <div class="">
              <div class="flex mt-6 items-center w-80 bigSmartphone:w-96">
                <img :src="item.article.imagePath" :alt="item.article.name" class="w-36 h-26 p-3 mr-6">
                <div class="details">
                  <h3 class="text-md font-semibold">{{ item.article.name }}</h3>
                  <p class="text-xs text-tMiddle">{{ item.article.feature }}</p>
                  <p class="mt-4 text-xs text-tGray">Price: ${{ item.article.price }}</p>
                  <p class="text-xs text-tGray">Available in store: {{ item.article.availableQuantity }}</p>
                  <p class="text-xs text-tGray">Quantity ordered: {{ item.quantity }}</p>
                </div>
              </div>
              <!-- REMOVE -->
              <div class="flex mt-2 justify-end w-full">
                <div class="flex justify-center items-center">
                  <button @click="removeItem(cartItem.idCart, item.article.idArticle)"
                    class="bg-danger text-sm text-white px-4 py-2 rounded-full hover:shadow-inner-strong">
                    Remove
                  </button>
                </div>
              </div>
            </div>
          </div>
        </div>
        <div v-if="cartItem.cartArticles.length > 0" class="flex flex-col items-end mx-6">
          <p class="text-md font-medium mb-2 text-right">Total Price: ${{ formatPrice(cartItem.totalPrice) }}</p>

          <button @click="createOrder(cartItem.idCart)"
            class="font-bold text-sm bg-primary text-white px-4 py-2 rounded-full hover:shadow-inner-strong">
            Confirm
          </button>
        </div>
      </div>

    </div>
  </div>
</template>
