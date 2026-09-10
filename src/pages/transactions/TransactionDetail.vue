<script setup lang="ts">
import { getTransaction } from '@/api/transactions.api';
import type { Transaction } from '@/types/transaction';
import { Button, Column, DataTable, useToast } from 'primevue';
import { computed, onMounted, ref } from 'vue';
import { useRoute } from 'vue-router';

const route = useRoute();
const toast = useToast();

const loading = ref(false);
const transaction = ref<Transaction | null>(null);

const transactionId = computed<number>(() => Number(route.params.id));

const formatCurrency = (val?: number | null) => {
  if (val === null || val === undefined) return 'Rp 0';
  return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 })
    .format(Number(val))
    .replace('IDR', 'Rp');
};

const formatDate = (dateStr?: string) => {
  if (!dateStr) return '-';
  const d = new Date(dateStr);
  const day = d.getDate();
  const months = [
    'Januari', 'Februari', 'Maret', 'April', 'Mei', 'Juni',
    'Juli', 'Agustus', 'September', 'Oktober', 'November', 'Desember'
  ];
  const month = months[d.getMonth()];
  const year = d.getFullYear();
  const hours = String(d.getHours()).padStart(2, '0');
  const minutes = String(d.getMinutes()).padStart(2, '0');
  return `${day} ${month} ${year} pukul ${hours}.${minutes}`;
};

const loadDetail = async () => {
  loading.value = true;
  try {
    const res = await getTransaction(transactionId.value);
    transaction.value = res.data.data;
  } catch (error: any) {
    toast.add({
      severity: 'error',
      summary: 'Error',
      detail: error.response?.data?.message || 'Failed to load transaction detail',
      life: 3000
    });
  } finally {
    loading.value = false;
  }
};

onMounted(() => {
  loadDetail();
});
</script>

<template>
  <div class="min-h-screen bg-surface-50 font-sans text-surface-900">
    <div class="flex justify-between items-center mb-8">
      <div>
        <h1 class="text-2xl font-bold text-surface-900 mb-1">
          Transaction Detail
        </h1>
        <p class="text-surface-500 text-sm">
          View transaction information
        </p>
      </div>

      <Button asChild v-slot="slotProps">
        <RouterLink :to="{ name: 'transactions' }" :class="slotProps?.class">
          <i class="pi pi-arrow-left"></i>
          Back
        </RouterLink>
      </Button>
    </div>

    <div v-if="loading" class="bg-white rounded-2xl border border-surface-200 p-12 text-center text-surface-400">
      <i class="pi pi-spin pi-spinner text-3xl mb-2"></i>
      <p>Loading transaction data...</p>
    </div>

    <div v-else-if="transaction" class="flex flex-col gap-6">
      <!-- Card 1: Transaction Information -->
      <div class="bg-white rounded-2xl border border-surface-200 p-6">
        <h2 class="text-base font-bold text-surface-900 mb-4">
          Transaction Information
        </h2>

        <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-4 gap-6">
          <div>
            <div class="text-xs text-surface-500 mb-1">Transaction Code</div>
            <div class="font-bold text-emerald-600">
              {{ transaction.code }}
            </div>
          </div>

          <div>
            <div class="text-xs text-surface-500 mb-1">Customer</div>
            <div class="font-semibold text-surface-900">
              {{ transaction.customer?.name || '-' }}
            </div>
          </div>

          <div>
            <div class="text-xs text-surface-500 mb-1">Date</div>
            <div class="text-surface-800 font-medium">
              {{ formatDate(transaction.created_at) }}
            </div>
          </div>

          <div>
            <div class="text-xs text-surface-500 mb-1">Total</div>
            <div class="font-bold text-emerald-600">
              {{ formatCurrency(transaction.total) }}
            </div>
          </div>
        </div>
      </div>

      <!-- Card 2: Items -->
      <div class="bg-white rounded-2xl border border-surface-200 p-6">
        <h2 class="text-base font-bold text-surface-900 mb-4">
          Items
        </h2>

        <DataTable :value="transaction.items || []" dataKey="id" class="clean-table">
          <template #empty>
            <div class="text-center py-6 text-surface-400">No items found</div>
          </template>

          <Column field="product" header="Product">
            <template #body="{ data }">
              <span class="font-semibold text-surface-900">
                {{ data.product?.name || data.product_name || `Product #${data.product_id}` }}
              </span>
            </template>
          </Column>

          <Column header="Price">
            <template #body="{ data }">
              <span class="text-surface-700 font-medium">
                {{ formatCurrency(data.price) }}
              </span>
            </template>
          </Column>

          <Column field="quantity" header="Qty" style="width: 6rem">
            <template #body="{ data }">
              <span class="text-surface-800 font-medium">
                {{ data.quantity }}
              </span>
            </template>
          </Column>

          <Column header="Subtotal" style="width: 10rem">
            <template #body="{ data }">
              <span class="font-bold text-surface-900">
                {{ formatCurrency(data.subtotal ?? (data.price * data.quantity)) }}
              </span>
            </template>
          </Column>
        </DataTable>
      </div>

      <!-- Card 3: Summary -->
      <div class="bg-white rounded-2xl border border-surface-200 p-6">
        <h2 class="text-base font-bold text-surface-900 mb-4">
          Summary
        </h2>

        <div class="flex flex-col items-end gap-3 text-sm">
          <div class="flex justify-between w-full max-w-xs text-surface-700">
            <span>Subtotal</span>
            <span class="font-bold text-surface-900">{{ formatCurrency(transaction.subtotal) }}</span>
          </div>

          <div class="flex justify-between w-full max-w-xs text-surface-700">
            <span>Tax</span>
            <span class="font-bold text-surface-900">{{ formatCurrency(transaction.tax) }}</span>
          </div>

          <div class="flex justify-between w-full max-w-xs text-surface-900 font-bold text-base pt-3 border-t border-surface-200">
            <span>Total</span>
            <span class="text-emerald-600 font-bold">{{ formatCurrency(transaction.total) }}</span>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
