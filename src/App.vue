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
  <div class="min-h-screen bg-gray-100 p-4">
    <div class="max-w-3xl mx-auto">

      <h1 class="text-2xl font-bold mb-4">💸 Urunan Villa</h1>

      <!-- Form -->
      <div class="bg-white p-4 rounded-xl shadow mb-4">
        <input v-model="name"
          placeholder="Nama"
          class="border p-2 w-full rounded mb-2" />

        <label class="flex items-center gap-2 mb-2">
          <input type="checkbox" v-model="withMeal" />
          Pakai Makan (+38K)
        </label>

        <button @click="add"
          class="bg-blue-500 text-white px-4 py-2 rounded w-full">
          Tambah
        </button>
      </div>

      <!-- List -->
      <div class="bg-white p-4 rounded-xl shadow mb-4">
        <h2 class="font-semibold mb-2">Daftar Peserta</h2>

        <div v-for="p in data" :key="p.id"
          class="flex justify-between items-center border-b py-2">

          <div>
            <p class="font-medium">{{ p.name }}</p>
            <p class="text-sm text-gray-500">
              Rp {{ p.amount.toLocaleString() }}
            </p>
          </div>

          <button
            v-if="p.amount === 100000"
            @click="upgrade(p.id)"
            class="bg-green-500 text-white px-2 py-1 rounded text-sm">
            +Makan
          </button>
        </div>
      </div>

      <!-- Total -->
      <div class="bg-white p-4 rounded-xl shadow">
        <p class="text-lg font-semibold">
          Total: Rp {{ total.toLocaleString() }}
        </p>
      </div>

    </div>
  </div>
</template>