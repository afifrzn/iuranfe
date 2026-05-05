<script setup>
import { ref, onMounted } from 'vue'
import axios from 'axios'

const API = import.meta.env.VITE_API_URL

const data = ref([])
const total = ref(0)
const name = ref('')
const withMeal = ref(false)

const fetchData = async () => {
  const res = await axios.get(`${API}/participants`)
  data.value = res.data.data
  total.value = res.data.total
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