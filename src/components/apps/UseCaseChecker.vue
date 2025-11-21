<script setup>
import Select from 'primevue/select';
import Checkbox from 'primevue/checkbox';
import InputText from 'primevue/inputtext';
import { Form } from '@primevue/forms'
import Card from 'primevue/card'
import Tree from 'primevue/tree';
import ServiceLabel from '../shared/ServiceLabel.vue';

import { ref, onMounted, Transition, nextTick } from 'vue';

// Form reactive vars
const brapps = ref([])
const useCases = ref([])
const formRef = ref(null)

// Form submit/response vars
const checkerResponse = ref(null);
const resultCard = ref(null);

const initialFormValues = ref({
  serverUrl: '',
  selectedBrapp: '',
  allUseCases: false,
  selectedUseCase: ''
});

// Fetch available BrApps from backend on component mount
onMounted(async () => {
  try {
    const response = await fetch('/brava/usecasechecker/apps')
    if (!response.ok) throw new Error('Failed to fetch brapps')
    const data = await response.json()
    brapps.value = data.map(app => ({ name: app, value: app }))

  } catch (err) {
    console.error(err)
  }
})

// Validation for form
const resolver = ({ values }) => {
  const errors = {};

  if (!values.serverUrl) {
    errors.serverUrl = [{ message: 'A server URL is required' }];
  } else {
    try {
      new URL(values.serverUrl)
    } catch (err) {
      errors.serverUrl = [{ message: 'Invalid URL provided' }]
    }
  }

  if (!values.selectedBrapp) {
    errors.selectedBrapp = [{ message: 'A selected BrApp is required' }]
  }

  if (!values.allUseCases && !values.selectedUseCase) {
    errors.selectedUseCase = [{ message: 'A selected Use Case is required' }]
  }

  return {
    values,
    errors
  };
};

const onSubmit = async ({ valid, values }) => {

  checkerResponse.value = null

  if (!valid) {
    return
  }

  const url = new URL('brava/usecasechecker/check', window.location.origin)
  url.searchParams.append('app', values?.selectedBrapp?.name)
  url.searchParams.append('useCase', values?.selectedUseCase?.name)
  url.searchParams.append('allUseCases', values?.allUseCases)
  url.searchParams.append('url', values?.serverUrl)

  const response = await fetch(url)
  if (!response.ok) throw new Error('Failed to fetch app use cases')

  const data = await response.json()

  checkerResponse.value = processCheckerResponse(data);

  await nextTick()
  resultCard.value?.$el?.focus()
}

const processCheckerResponse = (data) => {
  let useCasesAsTree = data.checkedUseCases.map((uc, ucIndex) => ({
    key: `${uc.useCaseName}-${ucIndex}`,
    label: "Use Case: " + `${uc.useCaseName}`,
    icon: uc.valid ? 'pi pi-fw pi-verified' : 'pi pi-fw pi-exclamation-triangle',
    children: uc.checkedEntities.map((entity, entityIndex) => ({
      key: `${entity.entityName}`,
      label: `${entity.entityName}`,
      icon: entity.valid ? 'pi pi-fw pi-verified' : 'pi pi-fw pi-exclamation-triangle',
      children: entity.checkedServices.map((service, serviceIndex) => ({
        key: `${service.service.serviceName}-${serviceIndex}`,
        serviceData: {
          method: service.service.methodRequired,
          version: service.service.versionRequired,
          name: service.service.serviceName,
          valid: service.valid
        },
        icon: service.valid ? 'pi pi-fw pi-check-circle' : 'pi pi-fw pi-times-circle',
        children: [{
          key: serviceIndex,
          label: service.message,
          isValid: service.valid
        }]
      }))
    }))
  }));

  return {
    appName: data.appName,
    serverUrl: data.serverUrl,
    checkedUseCases: useCasesAsTree
  }
};

const fetchUseCases = async (appName) => {
  // Reset use case picker form selection
  formRef.value.setFieldValue('selectedUseCase', null)

  try {
    const url = new URL('brava/usecasechecker/appusecases', window.location.origin)
    url.searchParams.append('app', appName)

    const response = await fetch(url)
    if (!response.ok) throw new Error('Failed to fetch app use cases')

    const data = await response.json()
    useCases.value = data.map(uc => ({ name: uc, value: uc }))

  } catch (err) {
    console.error(`Error fetching use cases for app [${appName}]:`, err)
    useCases.value = []
  }
}

</script>

<template>
  <div class="flex flex-col items-center w-full">

    <Form ref="formRef" v-slot="$form" @submit="onSubmit" :initialValues="initialFormValues" :resolver
      class="flex flex-col items-center w-full">

      <label for="serverUrl" class="mb-4 text-lg font-large text-surface-800 dark:text-surface-200 text-center">
        Provide a base server URL to test
      </label>
      <InputText id="serverUrl" type="url" name="serverUrl"
        class="w-full max-w-110 placeholder:text-surface-400 mb-2" />
      <div class="h-6">
        <Message v-if="$form.serverUrl?.invalid" severity="error" size="small" variant="simple">
          {{ $form?.serverUrl?.error?.message }}
        </Message>
      </div>

      <label for="brapp" class="mb-4 text-lg font-large text-surface-800 dark:text-surface-200 text-center mt-4">
        Choose a BrApp
      </label>
      <div class="flex flex-row items-center justify-center gap-4 w-full max-w-110 mb-2">
        <!-- Selecting a BrApp triggers a fetch for the use cases for that BrApp, populating selectedUseCase -->
        <Select inputId="brapp" name="selectedBrapp" :options="brapps" optionLabel="name" class="min-w-62"
          @change="fetchUseCases($form?.selectedBrapp?.value?.name)" />
        <label for="allUseCases"
          class="flex items-center  cursor-pointer text-surface-800 dark:text-surface-200 text-sm whitespace-nowrap">
          Check all use cases?
        </label>
        <Checkbox name="allUseCases" inputId="allUseCases" binary size="large" />
      </div>
      <div class="h-6">
        <Message v-if="$form.selectedBrapp?.invalid" severity="error" size="small" variant="simple">{{
          $form.selectedBrapp.error.message }}
        </Message>
      </div>

      <!-- Show use cases only if selectedUseCase is populated and the allUseCases checkbox is unchecked -->
      <div v-if="$form?.selectedBrapp?.value && !$form?.allUseCases?.value"
        class="w-full max-w-110 flex flex-col items-center">
        <label for="usecase" class="mb-4 text-lg font-large text-surface-800 dark:text-surface-200 text-center mt-4">
          Choose a Use Case
        </label>
        <Select name="selectedUseCase" :options="useCases" optionLabel="name" class="w-full max-w-110 mb-2" />
        <div class="h-6">
          <Message v-if="$form.selectedUseCase?.invalid" severity="error" size="small" variant="simple">{{
            $form?.selectedUseCase?.error?.message }}</Message>
        </div>
      </div>

      <Button type="submit" label="Submit" class="mt-4" />

      <!-- Results card with transition -->
      <Transition enter-active-class="transition duration-1000 ease-out" enter-from-class="opacity-0 translate-y-4"
        enter-to-class="opacity-100 translate-y-0" leave-active-class="transition duration-300 ease-in"
        leave-from-class="opacity-100 translate-y-0" leave-to-class="opacity-0 translate-y-4">
        <Card v-if="checkerResponse" :pt="{
          root: {
            tabindex: 0,
            class: 'focus-within:border-primary-400 focus-within:shadow-lg border-double'
          }
        }" ref="resultCard"
          class="mt-6 max-w-160 min-h-100 w-full rounded-xl border border-surface-200 dark:border-surface-400 text-center">
          <template #title>Use Cases Checked for {{ checkerResponse.appName }}</template>
          <template #content>
            <Tree :value="checkerResponse.checkedUseCases" class="mt-2" :pt="{
              nodeLabel: ({ context }) => {
                if (context.node.isValid) {
                  return {
                    class: 'text-green-500 dark:text-green-400 text-left ml-2 text-sm'
                  }
                } else if (context.node.isValid === false) {
                  return {
                    class: 'text-red-500 dark:text-red-400 text-left ml-2 text-sm'
                  }
                } else {
                  return {
                    class: ''
                  }
                }
              },
              nodeIcon: ({ context }) => ({
                class: context.node.icon?.includes('pi-times-circle') || context.node.icon?.includes('pi-exclamation-triangle') ? 'dark:!text-red-400 !text-red-500'
                  : context.node.icon?.includes('pi-check-circle') || context.node.icon?.includes('pi-verified') ? 'dark:!text-green-400 !text-green-500' : ''
              })
            }">
              <template #default="{ node }">
                <!-- If node has serviceData, use ServiceLabel component -->
                <ServiceLabel v-if="node.serviceData" :method="node.serviceData.method"
                  :version="node.serviceData.version" :name="node.serviceData.name" />
                <span v-else>{{ node.label }}</span>
              </template>
            </Tree>
          </template>
        </Card>
      </Transition>
    </Form>
  </div>
</template>