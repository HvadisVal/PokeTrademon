<template>
    <div class="dropdown absolute right-0 mt-2 w-40 bg-white rounded-lg shadow-lg border z-10 p-2">
      <ul class="text-sm text-gray-700 flex flex-col w-full">
        <div v-if="!state.isLoggedIn">
            <RouterLink to="/auth">
            <button class="btn-2 w-full text-center px-4 py-2">Log in</button>
            </RouterLink>
        </div>
  
        <RouterLink to="/register">
          <div v-if="!state.isLoggedIn">
              <li class="w-full px-4 py-2 underline cursor-pointer">Register now</li>
          </div>
        </RouterLink>
  
        <RouterLink to="/profile">
          <div v-if="state.isLoggedIn">
            <li class="w-full px-4 py-2 hover:bg-gray-100 cursor-pointer">Your collection</li>
          </div>  
        </RouterLink>

        <RouterLink to="/profile">
          <div v-if="state.isLoggedIn">
            <li class="w-full px-4 py-2 hover:bg-gray-100 cursor-pointer">Trades</li>
          </div>
        </RouterLink>
  
        <button
          v-if="state.isLoggedIn"
          @click="logout"
          class="w-full text-left px-4 py-2 underline"
        >
          Log out
        </button>
      </ul>
    </div>


    
  </template>
  

<script setup lang="ts">
import { useUsers } from '../../modules/auth/userModels';
import { state } from '../../modules/globalStates/state'

// fetch logout function
const { logout } = useUsers();
</script>

<style scoped>
.dropdown {
    border: 1px solid var(--dark-headline);
    width: 200px;
}

.dropdown ul {
    width: 100%;
}
</style>
  