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

onMounted(async () => {
  loading.value = true
  error.value = null
  try {
    const res = await fetch('https://api.instantwebtools.net/v1/passenger?page=0&size=10')
    if (!res.ok) throw new Error('Failed to fetch passengers')
    const data = await res.json()
    passengers.value = data.data
  } catch (err: any) {
    error.value = err.message || 'Unknown error'
  } finally {
    loading.value = false
  }
})
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
            <td style="padding: 0.5rem;">{{ p.name }}</td>
            <td style="padding: 0.5rem;">{{ p.trips }}</td>
            <td style="padding: 0.5rem;">
              <div v-for="a in p.airline" :key="a._id">
                <span>{{ a.name }}</span>
              </div>
            </td>
          </tr>
        </tbody>
      </table>
    </section>
  </main>
</template>
