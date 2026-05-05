<script setup>
import { ref, computed, onMounted } from 'vue'
import axios from 'axios'

const API = import.meta.env.VITE_API_URL || 'http://iuranapi.afifrzn.my.id/api'

const data = ref([])
const name = ref('')
const withMeal = ref(false)
const loadingBayar = ref(null)
const showSuccess = ref(false)

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

const launchConfetti = () => {
  const colors = ['#3b82f6', '#22c55e', '#f59e0b', '#ef4444', '#a855f7']
  for (let i = 0; i < 60; i++) {
    const el = document.createElement('div')
    el.style.cssText = `
      position: fixed;
      width: ${Math.random() * 8 + 6}px;
      height: ${Math.random() * 8 + 6}px;
      background: ${colors[Math.floor(Math.random() * colors.length)]};
      border-radius: ${Math.random() > 0.5 ? '50%' : '2px'};
      left: ${Math.random() * 100}vw;
      top: -10px;
      z-index: 9999;
      pointer-events: none;
      animation: confettiFall ${Math.random() * 1.5 + 1}s ease-in forwards;
      animation-delay: ${Math.random() * 0.5}s;
    `
    document.body.appendChild(el)
    setTimeout(() => el.remove(), 2500)
  }
}

const bayar = async (id) => {
  loadingBayar.value = id
  await axios.post(`${API}/participants/${id}/bayar`)
  loadingBayar.value = null
  showSuccess.value = true
  launchConfetti()
  setTimeout(() => showSuccess.value = false, 2000)
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

const downgrade = async (id) => {
  await axios.post(`${API}/participants/${id}/downgrade`)
  fetchData()
}

const search = ref('')

onMounted(fetchData)

const totalSemua = computed(() =>
  data.value.reduce((sum, p) => sum + p.amount, 0)
)

const totalBayar = computed(() =>
  data.value.filter(p => p.paid).reduce((sum, p) => sum + p.amount, 0)
)

const filtered = computed(() =>
  data.value.filter(p =>
    p.name.toLowerCase().includes(search.value.toLowerCase())
  )
)

const sisaBelumBayar = computed(() =>
  data.value.filter(p => !p.paid).reduce((sum, p) => sum + p.amount, 0)
)
</script>

<template>
  <div class="min-h-screen bg-gray-100 p-4 text-gray-800">
    <div class="max-w-3xl mx-auto">

      <h1 class="text-2xl font-bold mb-4">💸 Iuran Villa</h1>

      <!-- SEARCH -->
      <input
        v-model="search"
        placeholder="Cari nama..."
        class="border p-2 w-full rounded mb-3 text-black"
      />

      <!-- FORM TAMBAH -->
      <div class="bg-white p-4 rounded-xl shadow mb-4">
        <input
          v-model="name"
          placeholder="Nama"
          class="border p-2 w-full rounded mb-2 text-black"
        />
        <label class="flex items-center gap-2 mb-3 text-sm">
          <input type="checkbox" v-model="withMeal" />
          Pakai Makan (+38K)
        </label>
        <button
          @click="add"
          class="bg-blue-500 text-white px-4 py-2 rounded w-full"
        >
          Tambah
        </button>
      </div>

      <!-- TOTAL -->
<div class="grid grid-cols-3 gap-3 mb-4">
  <div class="bg-white p-3 rounded-xl shadow">
    <p class="text-sm text-gray-500">Total Harus</p>
    <p class="font-bold text-lg">
      Rp {{ new Intl.NumberFormat('id-ID').format(totalSemua) }}
    </p>
  </div>
  <div class="bg-green-100 p-3 rounded-xl shadow">
    <p class="text-sm text-gray-500">Sudah Bayar</p>
    <p class="font-bold text-lg text-green-700">
      Rp {{ new Intl.NumberFormat('id-ID').format(totalBayar) }}
    </p>
  </div>
  <div class="bg-red-100 p-3 rounded-xl shadow">
    <p class="text-sm text-gray-500">Kurang</p>
    <p class="font-bold text-lg text-red-600">
      Rp {{ new Intl.NumberFormat('id-ID').format(sisaBelumBayar) }}
    </p>
  </div>
</div>

      <!-- BELUM BAYAR -->
      <div class="bg-white p-4 rounded-xl shadow mb-4">
        <h2 class="font-semibold mb-2 text-red-500">Belum Bayar</h2>

        <div
          v-for="p in filtered.filter(p => !p.paid)"
          :key="p.id"
          class="flex justify-between border-b py-2"
        >
          <div>
            <p>{{ p.name }}</p>
            <p class="text-sm text-gray-500">
              Rp {{ new Intl.NumberFormat('id-ID').format(p.amount) }}
            </p>
          </div>

          <div class="flex gap-2">
            <button
              v-if="!p.with_meal"
              @click="upgrade(p.id)"
              class="bg-green-500 text-white px-2 py-1 rounded text-sm"
            >
              +Makan
            </button>
            <button
              v-if="p.with_meal"
              @click="downgrade(p.id)"
              class="bg-yellow-500 text-white px-2 py-1 rounded text-sm"
            >
              -Makan
            </button>
            <button
              @click="bayar(p.id)"
              :disabled="loadingBayar === p.id"
              class="bg-blue-500 text-white px-2 py-1 rounded text-sm flex items-center gap-1 transition-all disabled:opacity-70"
            >
              <span
                v-if="loadingBayar === p.id"
                class="inline-block w-3 h-3 border-2 border-white border-t-transparent rounded-full animate-spin"
              ></span>
              <span>{{ loadingBayar === p.id ? 'Proses...' : 'Bayar' }}</span>
            </button>
          </div>
        </div>
      </div>

      <!-- SUDAH BAYAR -->
      <div class="bg-white p-4 rounded-xl shadow">
        <h2 class="font-semibold mb-2 text-green-600">Sudah Bayar</h2>

        <div
          v-for="p in filtered.filter(p => p.paid)"
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
            <button
              v-if="p.with_meal"
              @click="downgrade(p.id)"
              class="bg-yellow-500 text-white px-2 py-1 rounded text-sm"
            >
              -Makan
            </button>
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

    <!-- SUCCESS MODAL -->
    <Transition name="pop">
      <div
        v-if="showSuccess"
        class="fixed inset-0 flex items-center justify-center z-50"
        style="background: rgba(0,0,0,0.3);"
      >
        <div class="bg-white rounded-2xl shadow-2xl p-8 flex flex-col items-center gap-3">
          <div class="success-circle">
            <svg class="checkmark" viewBox="0 0 52 52">
              <circle class="checkmark-circle" cx="26" cy="26" r="25" fill="none"/>
              <path class="checkmark-check" fill="none" d="M14 27l8 8 16-16"/>
            </svg>
          </div>
          <p class="text-lg font-bold text-gray-700">Pembayaran Berhasil!</p>
        </div>
      </div>
    </Transition>

  </div>
</template>

<style>
/* Pop transition */
.pop-enter-active { animation: popIn 0.35s cubic-bezier(0.34, 1.56, 0.64, 1); }
.pop-leave-active { animation: popOut 0.2s ease-in forwards; }
@keyframes popIn  { from { transform: scale(0.5); opacity: 0; } to { transform: scale(1); opacity: 1; } }
@keyframes popOut { from { transform: scale(1); opacity: 1; } to { transform: scale(0.5); opacity: 0; } }

/* Checkmark */
.success-circle { width: 80px; height: 80px; }
.checkmark { width: 80px; height: 80px; }

.checkmark-circle {
  stroke: #22c55e;
  stroke-width: 3;
  stroke-dasharray: 166;
  stroke-dashoffset: 166;
  animation: strokeCircle 0.5s ease forwards;
}

.checkmark-check {
  stroke: #22c55e;
  stroke-width: 4;
  stroke-linecap: round;
  stroke-linejoin: round;
  stroke-dasharray: 48;
  stroke-dashoffset: 48;
  animation: strokeCheck 0.3s ease 0.4s forwards;
}

@keyframes strokeCircle { to { stroke-dashoffset: 0; } }
@keyframes strokeCheck  { to { stroke-dashoffset: 0; } }

/* Confetti */
@keyframes confettiFall {
  0%   { transform: translateY(0) rotate(0deg); opacity: 1; }
  100% { transform: translateY(100vh) rotate(720deg); opacity: 0; }
}
</style>