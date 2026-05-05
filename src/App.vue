<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const API = import.meta.env.VITE_API_URL || 'http://10.10.101.243'

const data = ref([])
const total = ref(0)
const name = ref('')
const withMeal = ref(false)

const fetchData = async () => {
  try {
    const res = await axios.get(`${API}/participants`)
    data.value = res.data.data
    total.value = res.data.total
  } catch (err) {
    console.error(err)
  }
}

const add = async () => {
  await axios.post(`${API}/participants`, {
    name: name.value,
    with_meal: withMeal.value
  })
  name.value = ''
  fetchData()
}

const upgrade = async (id) => {
  await axios.post(`${API}/participants/${id}/upgrade`)
  fetchData()
}

onMounted(fetchData)
</script>

<template>
  <div style="padding: 20px">
    <h1>Urunan Villa</h1>

    <input v-model="name" placeholder="Nama" />
    <label>
      <input type="checkbox" v-model="withMeal" />
      Pakai Makan
    </label>
    <button @click="add">Tambah</button>

    <ul>
      <li v-for="p in data" :key="p.id">
        {{ p.name }} - {{ p.amount }}
        <button @click="upgrade(p.id)">Upgrade</button>
      </li>
    </ul>

    <h2>Total: {{ total }}</h2>
  </div>
</template>