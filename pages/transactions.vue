<script setup lang="ts">
import type {
  Account as IAccount,
  Category as ICategory,
  CategoryType as ICategoryType,
  SubCategory as ISubCategory,
  Transaction as ITransaction
} from "@prisma/client";

import {hide} from '~/plugins/modal';

import Button from '~/components/TheButton.vue';
import Modal from '~/components/Modal/TheModal.vue';
import ModalHeader from '~/components/Modal/TheModal.Header.vue';
import ModalBody from '~/components/Modal/TheModal.Body.vue';
import ModalFooter from '~/components/Modal/TheModal.Footer.vue';
import FormInput from '~/components/Form/FormInput.vue';
import Select from '~/components/Form/FormSelect.vue';
import EditRegistryIcon from '~/components/Icons/EditRegisttryIcon.vue';
import RemoveRegistryIcon from '~/components/Icons/RemoveRegisttryIcon.vue';

const transactionModalTarget: string = "accountModal"

const {data: accounts} = await useAsyncData<IAccount[]>('account', () => $fetch('/api/account'))
const {data: categoryTypes} = await useAsyncData<ICategoryType[]>('categoryTypes', () => $fetch('/api/category/type'))

const categories = ref<ICategory[]>([])
const categoryType = ref<ICategoryType>()
const subCategories = ref<ISubCategory[]>([])
const transactions = ref<ITransaction[]>([])

onMounted(() => {
  if (categoryTypes.value?.length) {
    categoryType.value = categoryTypes.value[0]
  }

  window.addEventListener('on-show-modal', async () => {
    const typeId = categoryType.value?.id ?? null
    const {data} = await useFetch<ICategory[]>(`/api/category/type/${typeId}`)

    categories.value = data.value as ICategory[]
  });
})

const handleSubmit = async (event: Event): Promise<void> => {
  const form = event.target as HTMLFormElement
  const formData = Object.fromEntries(new FormData(form))

  await useFetch('/api/transaction', {
    method: 'post',
    body: {
      ...formData,
      categoryTypeId: categoryType.value?.id
    }
  })

  form.reset()
  hide(transactionModalTarget)
}

const fetchSubCategory = async (event: Event) => {
  const {value} = event.target as HTMLSelectElement

  categoryType.value = categoryTypes.value?.find(item => item.id === value)
}

const selectSubCategory = async (event: Event) => {
  const category = event.target as HTMLSelectElement
  const categoryId = category.value

  const {data} = await useFetch<ISubCategory[]>(`/api/category/sub-category/${categoryId}`)

  subCategories.value = data.value as ISubCategory[]
}

watch(categoryType, async (category) => {
  if (category?.id) {
    const {data} = await useFetch<ITransaction[]>(`/api/transaction/categoryType/${category.id}`)

    transactions.value = data.value as ITransaction[]
  }
})
</script>

<template>
  <h1 class="text-gray-300 text-3xl font-bold">
    Transações
  </h1>

  <div class="flex justify-between items-center">
    <Select name="typeId" @change="fetchSubCategory" class="w-auto">
      <option v-for="{ id, description } in categoryTypes" :key="id" :value="id">
        {{ description }}
      </option>
    </Select>

    <div class="mt-4">
      <Button type="button" color="default" v-show-modal="transactionModalTarget">
        Adicionar
      </Button>
    </div>
  </div>

  <Modal ref="modalElement" :target="transactionModalTarget" position="top-center" class="mt-16">
    <ModalHeader :target="transactionModalTarget" :title="`Cadastrar transação - ${categoryType?.description}`"/>
    <form id="account" novalidate @submit.prevent="handleSubmit">
      <ModalBody :hasTitle="true">
        <div class="flex gap-4 items-end">
          <fieldset class="flex-1">
            <FormInput name="value" label="Valor"/>
          </fieldset>

          <fieldset class="flex-1">
            <FormInput name="date" type="date" label="Data"/>
          </fieldset>

          <fieldset class="basis-2/4">
            <Select name="accountId" label="Conta">
              <option v-for="{ id, name } in accounts" :key="id" :value="id">
                {{ name }}
              </option>
            </Select>
          </fieldset>
        </div>

        <fieldset class="flex-auto mt-6">
          <FormInput name="description" label="Descrição"/>
        </fieldset>

        <div class="flex mt-6 gap-4 items-end">
          <fieldset class="flex-1">
            <Select name="categoryId" label="Categoria" @change="selectSubCategory">
              <option selected>Selecione a categoria</option>
              <option v-for="{ id, name } in categories" :key="id" :value="id">
                {{ name }}
              </option>
            </Select>
          </fieldset>

          <fieldset class="flex-1">
            <Select name="subCategoryId" label="Sub Categoria">
              <option selected>Selecione a sub categoria</option>
              <option v-for="{ id, name } in subCategories" :key="id" :value="id">
                {{ name }}
              </option>
            </Select>
          </fieldset>
        </div>
      </ModalBody>
      <ModalFooter class="text-right">
        <Button type="submit" color="default">
          Confirmar
        </Button>
      </ModalFooter>
    </form>
  </Modal>

  <base-table v-if="transactions?.length" class="w-full text-sm text-left text-gray-400 mt-8" striped>
    <base-table-head class="text-xs text-gray-400 uppercase bg-gray-700">
      <base-table-head-cell>Name</base-table-head-cell>
      <base-table-head-cell><span class="sr-only">actions</span></base-table-head-cell>
    </base-table-head>
    <base-table-body>
      <base-table-body-row v-for="item in transactions" :key="item.description">
        <base-table-body-cell>
          {{ item.description }}
        </base-table-body-cell>
        <base-table-body-cell class="flex justify-end items-center gap-2">
          <Button class="bg-transparent">
            <EditRegistryIcon class="text-blue-600"/>
          </Button>
          <Button class="bg-transparent">
            <RemoveRegistryIcon class="text-red-500"/>
          </Button>
        </base-table-body-cell>
      </base-table-body-row>
    </base-table-body>
  </base-table>
</template>