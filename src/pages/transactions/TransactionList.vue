<script setup>
import { Button, Column, DataTable, IconField, InputIcon, InputText, Select } from 'primevue';
import { useTransactionStore } from '@/stores/transaction.store';
import { storeToRefs } from 'pinia';
import { onMounted } from 'vue';

import { useDebounceFn } from '@vueuse/core';

const transactionStore = useTransactionStore();
const { fetch, setLimit, setPage, nextPage, prevPage } = transactionStore;
const { items, loading, limit, currentPage, totalPages, search } = storeToRefs(transactionStore);

const onSearch = useDebounceFn(() => {
  setPage(1);
}, 400);

const formatCurrency = (val) => {
  if (val === null || val === undefined) return 'Rp 0';
  return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 })
    .format(Number(val))
    .replace('IDR', 'Rp');
};

const formatDate = (dateStr) => {
  if (!dateStr) return '-';
  const d = new Date(dateStr);
  const day = d.getDate();
  const months = ['Jan', 'Feb', 'Mar', 'Apr', 'Mei', 'Jun', 'Jul', 'Agu', 'Sep', 'Okt', 'Nov', 'Des'];
  const month = months[d.getMonth()];
  const year = d.getFullYear();
  const hours = String(d.getHours()).padStart(2, '0');
  const minutes = String(d.getMinutes()).padStart(2, '0');
  return `${day} ${month} ${year}, ${hours}.${minutes}`;
};

onMounted(() => {
  fetch();
});
</script>

<template>
  <div class="min-h-screen bg-surface-50 font-sans text-surface-900">
    <div class="flex justify-between items-center mb-8">
      <div>
        <h1 class="text-2xl font-bold text-surface-900 mb-1">
          Transactions
        </h1>
        <p class="text-surface-500 text-sm">
          The list here shows all transactions
        </p>
      </div>
    </div>

    <div class="bg-white rounded-2xl border border-surface-200 p-2">
      <div class="flex flex-col md:flex-row justify-between items-center px-4 py-4 gap-4">
        <IconField iconPosition="left" class="w-full md:w-80">
          <InputIcon class="pi pi-search text-surface-400" />
          <InputText v-model="search" placeholder="Search" @input="onSearch" />
        </IconField>
      </div>

      <DataTable :value="items" :loading="loading" dataKey="id" class="clean-table" :rowHover="true">
        <template #empty>
          <div class="text-center py-8 text-surface-500">
            No transactions found.
          </div>
        </template>

        <Column field="code" header="Code" class="min-w-[12rem]">
          <template #body="{ data }">
            <span class="font-bold text-emerald-600">
              {{ data.code }}
            </span>
          </template>
        </Column>

        <Column field="customer" header="Customer" class="min-w-[12rem]">
          <template #body="{ data }">
            <span class="font-medium text-surface-900">
              {{ data.customer?.name || '-' }}
            </span>
          </template>
        </Column>

        <Column field="total" header="Total">
          <template #body="{ data }">
            <span class="font-bold text-surface-900">{{ formatCurrency(data.total) }}</span>
          </template>
        </Column>

        <Column field="created_at" header="Date" class="min-w-[10rem]">
          <template #body="{ data }">
            <span class="text-surface-600">{{ formatDate(data.created_at) }}</span>
          </template>
        </Column>

        <Column header="Actions" style="width: 5rem">
          <template #body="{ data }">
            <div class="flex items-center gap-2">
              <RouterLink :to="{ name: 'transactions-detail', params: { id: data.id } }">
                <Button icon="pi pi-eye" text rounded severity="secondary"
                  class="w-9! h-9! border-surface-200! text-surface-300! hover:text-primary-600! hover:border-primary-600!" />
              </RouterLink>
            </div>
          </template>
        </Column>
      </DataTable>

      <div class="flex justify-between items-center px-4 py-4 border-t border-surface-100 gap-4">
        <div class="flex items-center gap-2">
          <span class="text-sm text-surface-500">Rows per page:</span>
          <Select :model-value="limit" :options="[5, 10, 20, 50]" @update:model-value="setLimit" />
        </div>

        <div class="flex items-center gap-4">
          <span class="text-sm font-medium text-surface-600">
            {{ currentPage }} of {{ totalPages }}
          </span>

          <div class="flex gap-1">
            <Button icon="pi pi-chevron-left" text rounded severity="secondary"
              :disabled="currentPage === 1"
              class="w-9! h-9! border! border-surface-500! hover:bg-surface-50!" @click="prevPage()" />
            <Button icon="pi pi-chevron-right" text rounded severity="secondary"
              :disabled="currentPage === totalPages"
              class="w-9! h-9! border! border-surface-500! hover:bg-surface-50!" @click="nextPage()" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
