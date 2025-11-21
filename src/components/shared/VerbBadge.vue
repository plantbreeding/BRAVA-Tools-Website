<script setup>
const props = defineProps({
    method: {
        type: String,
        required: true
    }
})

const colors = {
    GET: 'bg-green-100 text-green-800 border border-green-300',
    POST: 'bg-blue-100 text-blue-800 border border-blue-300',
    PUT: 'bg-yellow-100 text-yellow-800 border border-yellow-300',
    DELETE: 'bg-red-100 text-red-800 border border-red-300',
}

const badgeClass = colors[props.method.toUpperCase()] || 'bg-gray-100 text-gray-800 border border-gray-300'
</script>

<template>
    <span class="inline-flex items-center justify-center
           w-16 h-6 rounded-md font-semibold text-xs" :class="badgeClass">
        {{ method.toUpperCase() }}
    </span>
</template>