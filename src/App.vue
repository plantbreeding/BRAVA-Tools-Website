<script setup>
import AppFooter from "./components/main/AppFooter.vue";
import AppTopbar from "./components/main/AppHeader.vue";
import AppSelect from "./components/main/AppSelect.vue";
</script>

<template>
    <div class="bg-surface-50 dark:bg-surface-950 min-h-screen p-8 flex flex-col gap-6">
        <AppTopbar class="flex-shrink-0" />
        <main class="flex-1 flex justify-center">
            <AppSelect />
        </main>
        <AppFooter class="flex-shrink-0" />
    </div>
</template>
