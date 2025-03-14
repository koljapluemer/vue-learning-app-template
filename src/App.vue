<template>
  <div class="min-h-screen bg-base-200">
    <div class="navbar bg-base-100 shadow-lg">
      <div class="container mx-auto">
        <div class="flex-1">
          <router-link to="/" class="btn btn-ghost text-xl">Flashcard App</router-link>
        </div>
        <div class="flex-none">
          <ul class="menu menu-horizontal px-1">
            <li>
              <router-link to="/manage" class="btn btn-ghost">Manage Cards</router-link>
            </li>
            <li>
              <router-link to="/practice" class="btn btn-ghost">Practice</router-link>
            </li>
          </ul>
        </div>
      </div>
    </div>

    <RouterView />
  </div>
</template>

<script setup lang="ts">
import { RouterView } from 'vue-router';
</script>
