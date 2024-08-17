<script setup lang="ts">
import { computed, onMounted, ref, watchEffect } from 'vue'

type Patient = {
  city: string
  country: string
  patient_id: string
  year_of_birth: number | string
  gender: 'male' | 'female'
}

const page = ref(1)
const limit = ref(10)

const data = ref([])

const yearOfBirth = ref<string | number>('All')
const country = ref<string>('All')
const loading = ref<boolean>(false)
const errorMessage = ref<string | null>(null)
const searchText = ref<string>('')

const nextPage = () => {
  if (page.value * limit.value < data.value.length) page.value++
}

const prevPage = () => {
  if (page.value > 1) page.value--
}

const filterData = computed<Patient[]>(() => {
  return data.value
    .filter((item: Patient) => {
      if (yearOfBirth.value !== 'All') {
        if (item.year_of_birth !== +yearOfBirth.value) return false
      }
      if (searchText.value) {
        if (
          !item.city.toLowerCase().includes(searchText.value.toLowerCase()) &&
          !item.country.toLowerCase().includes(searchText.value.toLowerCase())
        )
          return false
      }
      return true
    })
    .slice((page.value - 1) * limit.value, page.value * limit.value)
})

watchEffect(async () => {
  loading.value = true
  try {
    const response = await fetch(
      `https://dev-963133558608530.api.raw-labs.com/patients-json${country.value === 'All' ? '' : `?country=${country.value}`}`
    )
    data.value = await response.json()
    loading.value = false
  } catch (error: any) {
    errorMessage.value = error?.message || 'An error occurred'
    loading.value = false
  }
})

onMounted(() => {
  console.log('HomeView mounted')
})
</script>

<template>
  <main class="container">
    <div class="top-table">
      <div class="toolbar">
        <div class="buttons">
          <button @click="prevPage">Prev</button>
          <button @click="nextPage">Next</button>
        </div>
        <input type="text" v-model="searchText" placeholder="Search by city or country..." />
      </div>
      <div class="filter">
        <select v-model="yearOfBirth">
          <option value="All">All</option>
          <option value="2024">2024</option>
          <option value="2023">2023</option>
          <option value="2022">2022</option>
          <option value="2021">2021</option>
          <option value="1994">1994</option>
          <option value="1944">1944</option>
          <option value="1933">1933</option>
        </select>
        <select v-model="country">
          <option value="All">All</option>
          <option value="Switzerland">Switzerland</option>
          <option value="France">France</option>
          <option value="Italy">Italy</option>
          <option value="Portugal">Portugal</option>
        </select>
      </div>
    </div>
    <div class="table">
      <table>
        <tr>
          <th>Patient ID</th>
          <th>Gender</th>
          <th>Year of birth</th>
          <th>City</th>
          <th>Country</th>
        </tr>
        <template v-if="loading">
          <p>Loading...</p>
        </template>
        <template v-else-if="errorMessage">
          <p>{{ errorMessage }}</p>
        </template>
        <template v-else>
          <tr v-for="item in filterData" :key="item.patient_id">
            <td>{{ item.patient_id }}</td>
            <td>{{ item.gender }}</td>
            <td>{{ item.year_of_birth }}</td>
            <td>{{ item.city }}</td>
            <td>{{ item.country }}</td>
          </tr>
        </template>
      </table>
    </div>
  </main>
</template>

<style scope>
.container {
  margin: 0 auto;
  width: 80%;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}
.buttons,
.toolbar {
  display: flex;
  align-items: center;
  gap: 8px;
}
.toolbar input {
  outline: none;
  height: 26px;
}
.buttons button {
  background-color: #181818;
  color: #fff;
  outline: none;
  box-shadow: none;
  border: 1px solid #ddd;
  padding: 4px 8px;
  border-radius: 4px;
  transition: all 0.3s;
}
.buttons button:hover {
  background-color: #fff;
  color: #181818;
  outline: none;
  box-shadow: none;
  border: 1px solid #ddd;
  padding: 4px 8px;
  border-radius: 4px;
}
.top-table {
  display: flex;
  justify-content: space-between;
  gap: 1rem;
}
.filter {
  display: flex;
  gap: 0.5rem;
}
select {
  padding: 0.5rem;
  border-radius: 5px;
  border: 1px solid #ddd;
  background-color: #181818;
  color: #fff;
}
table {
  width: 100%;
  border-collapse: collapse;
}
th,
td {
  border: 1px solid #ddd;
  padding: 8px;
  text-align: left;
}
th {
  font-weight: 700;
  color: #00bd7e;
  text-align: center;
}
</style>
