<script setup>
import VerbBadge from './VerbBadge.vue'

defineProps({
    method: { type: String, required: true },
    version: { type: [String, Number], required: true },
    name: { type: String, required: true }
})
</script>

<template>
    <div class="inline-flex items-center space-x-2 ml-2">
        <VerbBadge :method="method" />
        <span class="text-sm font-medium text-surface-700 dark:text-surface-200"> /{{ name }}</span>
        <span
            class="font-semibold text-sm w-10 text-center border bg-gray-100 border-gray-400 text-gray-800 dark:bg-gray-700 dark:border-gray-500 dark:text-gray-200 rounded-sm px-1 py-[1px] tracking-wide">V{{
                version
            }}</span>
    </div>
</template>