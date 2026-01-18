<template>

  <div class="app">
    <div class="wrapper">
      <button class="btn" type="button" @click="loadData">
        Load users
      </button>

      <p v-if="error" class="error">Error</p>
      <p v-if="loading" class="loading">Loading...</p>

      <input
          v-model="usersSearch"
          type="text"
          placeholder="Search users"
      />

      <label class="checkbox">
        <input type="checkbox" v-model="onlyWithCompany"/>
        Only users with company
      </label>

      <select v-model="sortOrder">
        <option value="">Default</option>
        <option value="asc">A–Z</option>
        <option value="desc">Z–A</option>
      </select>
      <button class="reset" type="button" @click="resetFilters">
        Reset filters
      </button>
    </div>
    <ul>
      <li v-for="user in filteredUsers" :key="user.id">
        <strong>{{ user.name }}</strong>
        <span>{{ user.email }}</span>
      </li>
    </ul>
  </div>

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

.app {
  max-width: 1140px;
  margin: 40px auto;
  padding: 20px;
  font-family: Arial, sans-serif;
}

.wrapper {
  width: 250px;
  margin: 0 auto;
}


.btn {
  padding: 10px 16px;
  background-color: #4f46e5;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  margin-bottom: 16px;
}

input,
select {
  width: 100%;
  padding: 8px 10px;
  margin-bottom: 12px;
  border: 1px solid #ccc;
  border-radius: 6px;
}

.checkbox {
  display: flex;
  align-items: center;
  gap: 8px;
  margin-bottom: 12px;
}

ul {
  list-style: none;
  padding: 0;
  margin: 20px 0;
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

li {
  width: 200px;
  padding: 10px;
  border-radius: 10px;
  box-shadow: 0 4px 8px 0 rgba(0, 0, 0, 0.2), 0 6px 20px 0 rgba(0, 0, 0, 0.19);
}

li span {
  display: block;
  font-size: 14px;
  color: #555;
}

.error {
  color: red;
}

.loading {
  color: gray;
}

.reset {
  margin-top: 12px;
  padding: 8px 14px;
  background-color: #e5e7eb;
  border: none;
  border-radius: 6px;
  cursor: pointer;
}
</style>

