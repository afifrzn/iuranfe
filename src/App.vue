<script setup>
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'

const API = import.meta.env.VITE_API_URL || 'http://iuranapi.afifrzn.my.id/api'

const data = ref([])
const name = ref('')
const withMeal = ref(false)

const fetchData = async () => {
  const res = await axios.get(`${API}/participants`)
  data.value = res.data.data
}

const add = async () => {
  if (!name.value) return

  await axios.post(`${API}/participants`, {
    name: name.value,
    with_meal: withMeal.value
  })

  name.value = ''
  withMeal.value = false
  fetchData()
}

const bayar = async (id) => {
  await axios.post(`${API}/participants/${id}/bayar`)
  fetchData()
}

const upgrade = async (id) => {
  await axios.post(`${API}/participants/${id}/upgrade`)
  fetchData()
}

const batal = async (id) => {
  if (!confirm('Yakin batalin pembayaran?')) return
  await axios.post(`${API}/participants/${id}/batal`)
  fetchData()
}

const search = ref('')

const downgrade = async (id) => {
  await axios.post(`${API}/participants/${id}/downgrade`)
  fetchData()
}

onMounted(fetchData)

const totalSemua = computed(() =>
  data.value.reduce((sum, p) => sum + p.amount, 0)
)

const totalBayar = computed(() =>
  data.value.filter(p => p.paid)
            .reduce((sum, p) => sum + p.amount, 0)
)

const belumBayar = computed(() =>
  data.value.filter(p => !p.paid)
)

const sudahBayar = computed(() =>
  data.value.filter(p => p.paid)
)

const filtered = computed(() => {
  return data.value.filter(p =>
    p.name.toLowerCase().includes(search.value.toLowerCase())
  )
})
</script>

<template>
  <div class="min-h-screen bg-gray-100 p-4 text-gray-800">
    <div class="max-w-3xl mx-auto">

      <h1 class="text-2xl font-bold mb-4">💸 Iuran Villa</h1>

      <!-- FORM -->
       <input
        v-model="search"
        placeholder="Cari nama..."
        class="border p-2 w-full rounded mb-3 text-black"
      />
      <div class="bg-white p-4 rounded-xl shadow mb-4">
        <input v-model="name" placeholder="Nama"
          class="border p-2 w-full rounded mb-2 text-black" />

        <label class="flex items-center gap-2 mb-3 text-sm">
          <input type="checkbox" v-model="withMeal" />
          Pakai Makan (+38K)
        </label>

        <button @click="add"
          class="bg-blue-500 text-white px-4 py-2 rounded w-full">
          Tambah
        </button>
      </div>

      <!-- TOTAL -->
      <div class="grid grid-cols-2 gap-3 mb-4">
        <div class="bg-white p-3 rounded-xl shadow">
          <p class="text-sm">Total Harus</p>
          <p class="font-bold text-lg">
            Rp {{ new Intl.NumberFormat('id-ID').format(totalSemua) }}
          </p>
        </div>

        <div class="bg-green-100 p-3 rounded-xl shadow">
          <p class="text-sm">Sudah Bayar</p>
          <p class="font-bold text-lg text-green-700">
            Rp {{ new Intl.NumberFormat('id-ID').format(totalBayar) }}
          </p>
        </div>
      </div>

      <!-- BELUM BAYAR -->
      <div class="bg-white p-4 rounded-xl shadow mb-4">
        <h2 class="font-semibold mb-2 text-red-500">Belum Bayar</h2>

        <div v-for="p in filtered.filter(p => !p.paid)" :key="p.id"
          class="flex justify-between border-b py-2">

          <div>
            <p>{{ p.name }}</p>
            <p class="text-sm text-gray-500">
              Rp {{ new Intl.NumberFormat('id-ID').format(p.amount) }}
            </p>
          </div>

          <div class="flex gap-2">
            <!-- +Makan -->
            <button
              v-if="!p.with_meal"
              @click="upgrade(p.id)"
              class="bg-green-500 text-white px-2 py-1 rounded text-sm"
            >
              +Makan
            </button>
            <!-- -Makan -->
            <button
              v-if="p.with_meal"
              @click="downgrade(p.id)"
              class="bg-yellow-500 text-white px-2 py-1 rounded text-sm"
            >
              -Makan
            </button>
            <!-- Bayar -->
            <button
              @click="bayar(p.id)"
              class="bg-blue-500 text-white px-2 py-1 rounded text-sm"
            >
              Bayar
            </button>
          </div>
        </div>
      </div>

      <!-- SUDAH BAYAR -->
      <div class="bg-white p-4 rounded-xl shadow">
        <h2 class="font-semibold mb-2 text-green-600">Sudah Bayar</h2>

        <div v-for="p in filtered.filter(p => p.paid)"
          :key="p.id"
          class="flex justify-between border-b py-2"
        >
          <div>
            <p class="text-green-700">{{ p.name }}</p>
            <p class="text-sm text-gray-500">
              Rp {{ new Intl.NumberFormat('id-ID').format(p.amount) }}
            </p>
          </div>

          <div class="flex gap-2">
            <!-- -Makan kalau sudah makan -->
            <button
              v-if="p.with_meal"
              @click="downgrade(p.id)"
              class="bg-yellow-500 text-white px-2 py-1 rounded text-sm"
            >
              -Makan
            </button>
            <!-- +Makan kalau belum -->
            <button
              v-if="!p.with_meal"
              @click="upgrade(p.id)"
              class="bg-green-500 text-white px-2 py-1 rounded text-sm"
            >
              +Makan
            </button>
            <button
              @click="batal(p.id)"
              class="bg-red-500 text-white px-2 py-1 rounded text-sm"
            >
              Batal
            </button>
          </div>
        </div>
      </div>

    </div>
  </div>
</template>