<script setup>
import { ref, computed } from 'vue'
import UseCaseChecker from '../apps/UseCaseChecker.vue'
import BrapiValidator from '../apps/BrapiValidator.vue'
import { KeepAlive } from 'vue'

// Add apps here to create new buttons
const apps = [
  {
    id: 'brapiValidator',
    name: 'BrAPI Validator',
    component: BrapiValidator,
    icon: 'pi pi-check-circle',
    description: 'Validate your BrAPI server endpoints'
  },
  {
    id: 'useCaseChecker',
    name: 'Use Case Checker',
    component: UseCaseChecker,
    icon: 'pi pi-search',
    description: 'Validate use case workflows on a server'
  }
]

const activeApp = ref(null)

const selectApp = (app) => {
  activeApp.value = app
}

const activeAppInfo = computed(() =>
  apps.find(a => a.id == activeApp.value)
)

const activeAppComponent = computed(() =>
  activeAppInfo.value?.component || null
)

</script>

<template>
  <div
    class="flex flex-col flex-1 bg-surface-0 dark:bg-surface-900 px-6 py-10 rounded-2xl max-w-7xl border border-surface-300 dark:border-surface-700 w-full">

    <!-- Header + Back Button -->
    <div class="flex items-center justify-between mb-4" v-if="activeApp">
      <Button icon="pi pi-arrow-left" label="Back" text @click="activeApp = null" class="text-primary" />
      <h2 class="text-xl font-medium">{{ activeAppInfo?.name }}</h2>
      <span class="w-20"></span>
    </div>

    <!-- App Select -->
    <div v-if="!activeApp" class="flex flex-col items-center space-y-8 flex-1">
      <h1 class="text-2xl font-bold text-center">Choose a BRAVA Tool</h1>

      <div class="grid grid-cols-1 sm:grid-cols-2 gap-6 w-full max-w-lg">
        <div v-for="app in apps" :key="app.id" class="p-6 rounded-2xl border border-surface-200 dark:border-surface-700
                    hover:bg-surface-50 dark:hover:bg-surface-800 cursor-pointer
                    text-center transition transform hover:-translate-y-1 hover:shadow-md" @click="selectApp(app.id)">
          <i :class="`${app.icon} text-4xl text-primary mb-3`"></i>
          <h2 class="text-xl font-semibold mb-2">{{ app.name }}</h2>
          <p class="text-sm text-surface-600 dark:text-surface-300">{{ app.description }}</p>
        </div>
      </div>
    </div>

    <!-- Handle Active App -->
    <div v-show="activeApp" class="flex-1">
      <KeepAlive>
        <component :is="activeAppComponent" />
      </KeepAlive>
    </div>
  </div>
</template>
