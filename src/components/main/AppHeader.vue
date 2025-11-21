<script setup>
import { useLayout } from "../../composables/useLayout";

const { isDarkMode, toggleDarkMode } = useLayout();
</script>

<template>
  <div
    class="bg-surface-0 dark:bg-surface-900 px-6 py-10 rounded-2xl max-w-7xl mx-auto border border-surface-300 dark:border-surface-700 w-full">
    <div class="flex justify-between items-center">
      <div class="flex gap-3 items-center">
        <a href="https://brapi.org" target="_blank" rel="noopener noreferrer">
          <img src="/src/assets/icons/brapi-logo.svg" alt="BrAPI Logo"
            class="h-4 sm:h-6 md:h-8 w-auto transition-all duration-300" />
        </a>
      </div>

      <div class="absolute left-1/2 transform -translate-x-1/2 text-center">
        <h1 class="text-xl mb-2">
          BrAPI Validation (BRAVA) Tools
        </h1>

        <ul class="list-disc list-inside text-sm space-y-1 text-surface-600 dark:text-surface-300 hidden lg:block">
          <li>Validate BrAPI-compatible servers based off of <a
              href="https://app.swaggerhub.com/apis-docs/PlantBreedingAPI/BrAPI-Core/2.1"
              class="hover:text-primary underline">existing spec</a></li>
          <li>Test available BrApp use cases against a server, or create your own</li>
        </ul>
      </div>

      <div class="flex items-center gap-2">
        <button type="button"
          class="w-10 h-10 flex items-center justify-center rounded-full hover:bg-surface-100 dark:hover:bg-surface-800 transition-all text-surface-900 dark:text-surface-0 focus-visible:outline-hidden focus-visible:ring-1 focus-visible:ring-primary focus-visible:ring-offset-2 focus-visible:ring-offset-surface-0 dark:focus-visible:ring-offset-surface-950"
          @click="toggleDarkMode">
          <i :class="['pi text-base', { 'pi-moon': !isDarkMode, 'pi-sun': isDarkMode }]" />
        </button>
      </div>
    </div>
  </div>
</template>
