<template>
  <button type="submit" @click="loadData">Load users</button>
  <p v-if="error">Error</p>
  <p v-if="loading">loading...</p>
  <input v-model="usersSearch" placeholder="Search">
  <label>
    <input type="checkbox" v-model="onlyWithCompany"/>
    Only users with company
  </label>
  <select v-model="sortOrder">
    <option value="">Default</option>
    <option value="asc">A–Z</option>
    <option value="desc">Z–A</option>
  </select>
  <ul>
    <li v-for="user in filteredUsers" :key="user.id">
      {{ user.name }} - {{ user.email }}
    </li>
  </ul>
  <button type="button" @click="resetFilters">
    Reset filters
  </button>
</template>

<script setup>
import {computed, ref, watch} from "vue";

const loading = ref(false);
const error = ref(null);
const allUsers = ref([]);
const usersSearch = ref('');
const debouncedSearch = ref("");
let timeoutId = null;
const onlyWithCompany = ref(false);
let sortOrder = ref('');


async function loadData() {
  loading.value = true;
  error.value = null;

  try {
    const res = await fetch("https://jsonplaceholder.typicode.com/users")
    const data = await res.json();
    allUsers.value = data;
  } catch (err) {
    error.value = "Error fetching users.";
  } finally {
    loading.value = false;
  }
}

watch(usersSearch, (newValue) => {
  if (!allUsers.value.length) return;

  clearTimeout(timeoutId);
  timeoutId = setTimeout(() => {
    debouncedSearch.value = newValue;
  }, 300);
});


const filteredUsers = computed(() => {
  let result = allUsers.value;

  if (onlyWithCompany.value) {
    result = result.filter(
        (user) => user.company && user.company.name
    );
  }

  if (debouncedSearch.value) {
    const text = debouncedSearch.value.toLowerCase();
    result = result.filter(
        (user) =>
            user.name.toLowerCase().includes(text) ||
            user.email.toLowerCase().includes(text)
    );
  }

  if (sortOrder.value) {
    result = [...result].sort((a, b) => {
      return sortOrder.value === 'asc'
          ? a.name.localeCompare(b.name)
          : b.name.localeCompare(a.name);
    });
  }
  return result;
});

function resetFilters() {
  usersSearch.value = '';
  debouncedSearch.value = '';
  onlyWithCompany.value = false;
  sortOrder.value = '';
}
</script>


<style scoped>


</style>
