<template>
  <div class="fixed top-0 left-0 h-full w-full bg-black z-10 opacity-70"></div>
  <div class="bg-white w-96 h-full fixed right-0 top-0 z-20 p-8">
    <DrawerHead />

    <div v-if="!totalPrice || orderId" class="flex h-full items-center">
      <InfoBlock
        v-if="!totalPrice && !orderId"
        title="Корзина пустая"
        description="Добавьте хотя бы одну пару кроссовок, чтобы сделать заказ."
        image-url="/package-icon.png"
      />
      <InfoBlock
        v-if="orderId"
        title="Заказ оформлен"
        :description="`Ваш заказ #1${orderId} скоро будет передан курьерской доставке`"
        image-url="/order-success-icon.png"
      />
    </div>

    <div v-else>
      <CartListItem />

      <div class="flex flex-col gap-4 my-7">
        <div class="flex gap-2">
          <span>Итого:</span>
          <div class="flex-1 border-b border-dashed"></div>
          <b>{{ totalPrice }} &#8381;</b>
        </div>

        <div class="flex gap-2">
          <span>Налог 5%:</span>
          <div class="flex-1 border-b border-dashed"></div>
          <b>{{ vatPrice }} &#8381;</b>
        </div>

        <button
          :disabled="buttonDisabled"
          @click="createOrder"
          class="bg-lime-500 w-full rounded-xl py-3 text-white hover:bg-lime-600 transition active:bg-lime-700 disabled:bg-slate-300 cursor-pointer mt-4"
        >
          Оформить заказ
        </button>
      </div>
    </div>
  </div>
</template>

<script setup>
import axios from 'axios'
import { ref, inject, watch, computed } from 'vue'
import DrawerHead from './DrawerHead.vue'
import CartListItem from './CartListItem.vue'
import InfoBlock from './InfoBlock.vue'

const props = defineProps({
  totalPrice: Number,
  vatPrice: Number,
})
const isCreatingOrder = ref(false)
const orderId = ref(null)
const { cart, closeDrawer } = inject('cart')

//функция, которая переносит заказ на бэкенд
const createOrder = async () => {
  try {
    isCreatingOrder.value = true
    const { data } = await axios.post(`https://780569a4959bcd37.mokky.dev/orders`, {
      items: cart.value,
      totalPrice: props.totalPrice.value,
    })

    cart.value = []

    orderId.value = data.id
  } catch (err) {
    console.log(err)
  } finally {
    isCreatingOrder.value = false
  }
}
const cartIsEmpty = computed(() => cart.value.length === 0)
const buttonDisabled = computed(() => isCreatingOrder.value || cartIsEmpty.value)
</script>
