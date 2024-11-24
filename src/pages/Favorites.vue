<template>
  <div>
    <h2 class="text-3xl font-bold mb-8">Мои закладки</h2>

    <CardList :items="favorites" is-favorites />
  </div>
</template>

<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'
import CardList from '../components/CardList.vue'

const favorites = ref([])

onMounted(async () => {
  try {
    const { data } = await axios.get(
      `https://780569a4959bcd37.mokky.dev/favorites?_relations=items`,
    )
    favorites.value = data.map((obj) => obj.item)
  } catch (err) {
    console.log(err)
  }
})
</script>
