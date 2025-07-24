<script setup lang="ts">
import { ref, onMounted } from 'vue'
import TheWelcome from '../components/TheWelcome.vue'

interface Airline {
  _id: string;
  name: string;
  country: string;
  logo: string;
  slogan: string;
  head_quaters: string;
  website: string;
  established: string;
}

interface Passenger {
  _id: string;
  name: string;
  trips: number;
  airline: Airline[];
}

const passengers = ref<Passenger[]>([])
const loading = ref(true)
const error = ref<string | null>(null)
const selectedPassenger = ref<Passenger | null>(null)
const detailLoading = ref(false)
const detailError = ref<string | null>(null)
const CORS_PROXY = 'https://corsproxy.io/?';

onMounted(async () => {
  loading.value = true
  error.value = null
  try {
    const res = await fetch(CORS_PROXY + encodeURIComponent('https://api.instantwebtools.net/v1/passenger?page=0&size=10'))
    if (!res.ok) throw new Error('Failed to fetch passengers')
    const data = await res.json()
    passengers.value = data.data
  } catch (err: any) {
    error.value = err.message || 'Unknown error'
  } finally {
    loading.value = false
  }
})

async function showPassengerDetails(id: string) {
  selectedPassenger.value = null
  detailLoading.value = true
  detailError.value = null
  try {
    const res = await fetch(CORS_PROXY + encodeURIComponent(`https://api.instantwebtools.net/v1/passenger/${id}`))
    if (!res.ok) throw new Error('Failed to fetch passenger details')
    const data = await res.json()
    selectedPassenger.value = data
  } catch (err: any) {
    detailError.value = err.message || 'Unknown error'
  } finally {
    detailLoading.value = false
  }
}
</script>

<template>
  <main>
    <TheWelcome />
    <section style="margin-top: 2rem;">
      <h2>Passenger List</h2>
      <div v-if="loading">Loading passengers...</div>
      <div v-else-if="error" style="color: red;">Error: {{ error }}</div>
      <table v-else style="width: 100%; border-collapse: collapse;">
        <thead>
          <tr>
            <th style="border-bottom: 1px solid #ccc; text-align: left; padding: 0.5rem;">Name</th>
            <th style="border-bottom: 1px solid #ccc; text-align: left; padding: 0.5rem;">Trips</th>
            <th style="border-bottom: 1px solid #ccc; text-align: left; padding: 0.5rem;">Airline</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="p in passengers" :key="p._id">
            <td style="padding: 0.5rem; cursor: pointer; color: #1976d2; text-decoration: underline;" @click="showPassengerDetails(p._id)">{{ p.name }}</td>
            <td style="padding: 0.5rem;">{{ p.trips }}</td>
            <td style="padding: 0.5rem;">
              <div v-for="a in p.airline" :key="a._id">
                <span>{{ a.name }}</span>
              </div>
            </td>
          </tr>
          <tr v-if="selectedPassenger && !detailLoading && !detailError" :key="'details-' + selectedPassenger._id">
            <td colspan="3" style="background: #f9f9f9; padding: 1rem;">
              <strong>Passenger Details:</strong><br>
              Name: {{ selectedPassenger.name }}<br>
              Trips: {{ selectedPassenger.trips }}<br>
              <div v-for="a in selectedPassenger.airline" :key="a._id" style="margin-top: 0.5rem;">
                <strong>Airline:</strong> {{ a.name }}<br>
                Country: {{ a.country }}<br>
                Slogan: {{ a.slogan }}<br>
                Website: <a :href="a.website" target="_blank">{{ a.website }}</a><br>
              </div>
            </td>
          </tr>
          <tr v-if="detailLoading">
            <td colspan="3" style="background: #f9f9f9; padding: 1rem; color: #888;">Loading passenger details...</td>
          </tr>
          <tr v-if="detailError">
            <td colspan="3" style="background: #f9f9f9; padding: 1rem; color: red;">Error: {{ detailError }}</td>
          </tr>
        </tbody>
      </table>
    </section>
  </main>
</template>
