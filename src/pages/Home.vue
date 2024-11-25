<template>
  <div class="flex justify-between items-center">
    <h2 class="text-3xl font-bold mb-8">Все кроссовки</h2>

    <div class="flex gap-4">
      <select class="py-2 px-3 border rounded-md outline-none" @change="onChangeSelect">
        <option value="name">По названию</option>
        <option value="price">По цене (дешевые)</option>
        <option value="-price">По цене (дорогие)</option>
      </select>
      <div class="relative">
        <img src="/search.svg" alt="Иконка поиск" class="absolute left-4 top-3" />
        <input
          type="text"
          placeholder="Поиск..."
          class="border rounded-md py-2 pl-11 pr-4 outline-none focus:border-gray-400"
          @change="onChangeSearchInput"
        />
      </div>
    </div>
  </div>

  <div class="mt-10">
    <CardList :items="items" @addToFavorite="addToFavorite" @add-to-cart="onClickAddPlus" />
  </div>
</template>

<script setup>
import axios from 'axios'
import { reactive, watch, ref, onMounted } from 'vue'
import { inject } from 'vue'
import CardList from '../components/CardList.vue'
import debounce from 'lodash.debounce'

const { cart, addToCart, removeFromCart } = inject('cart')

//стейт, который хранит наши товары(кроссовки)
//ref использовали, потому что он хранит массив
const items = ref([])

//стейт, который хранит наши фильтры, reacrive - для объектов
const filters = reactive({
  sortBy: '',
  searchQuery: '',
})

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}
//две функции, которые следят за изменениями селекта и поиска(input)
const onChangeSelect = (event) => {
  filters.sortBy = event.target.value
}

const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value
}, 350)

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
        item_id: item.id,
      }
      item.isFavorite = true

      const { data } = await axios.post(`https://780569a4959bcd37.mokky.dev/favorites`, obj)

      item.favoriteId = data.id
    } else {
      item.isFavorite = false
      await axios.delete(`https://780569a4959bcd37.mokky.dev/favorites/${item.favoriteId}`)
      item.isFavorite = null
    }
  } catch (err) {
    console.log(err)
  }
}

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(`https://780569a4959bcd37.mokky.dev/favorites`)
    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.item_id === item.id)
      if (!favorite) {
        return item
      }

      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id,
      }
    })
  } catch (err) {
    console.log(err)
  }
}

//функция, которая выполняет запрос на бэкенд, при изменениях в фильтре и при первом рендере
const fetchItems = async () => {
  try {
    //тут по умолчанию всегда передаем сортировку
    const params = {
      sortBy: filters.sortBy,
    }

    //но при этом проверяем изменился ли параметр searchQuery
    if (filters.searchQuery) {
      params.title = `*${filters.searchQuery}*`
    }

    //делаем запрос на бэкенд, передаем параметры - params
    const { data } = await axios.get(`https://780569a4959bcd37.mokky.dev/items`, {
      params,
    })

    //после того, как пришел ответ вшиваем его в title
    //далее рендерим это с помощью CardList, в который передали items (передаем туда пропс)
    //и уже там в CardList с помощью директивы v-for рендерятся карточки
    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false,
    }))
  } catch (err) {
    console.log(err)
  }
}

onMounted(async () => {
  const localCart = localStorage.getItem('cart')
  cart.value = localCart ? JSON.parse(localCart) : []

  await fetchItems()
  await fetchFavorites()

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id),
  }))
})

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false,
  }))
})

watch(filters, fetchItems)
</script>
