<script setup>
import { Button, Column, ConfirmDialog, DataTable, IconField, InputIcon, InputText, Select, useConfirm, useToast } from 'primevue';
import { useProductStore } from '@/stores/product.store';
import { getCategories } from '@/api/product-categories.api';
import { storeToRefs } from 'pinia';
import { computed, onMounted, ref } from 'vue';

import { useDebounceFn } from '@vueuse/core';
import { deleteProduct } from '@/api/products.api';

const productStore = useProductStore();
const { fetch, setLimit, setPage, setCategory, nextPage, prevPage } = productStore;
const { items, loading, limit, currentPage, totalPages, search, categoryId } = storeToRefs(productStore);

const confirm = useConfirm();
const toast = useToast();
const categories = ref([]);

const categoryOptions = computed(() => [
  { id: null, name: 'All Categories' },
  ...categories.value
]);

const displayedItems = computed(() => {
  if (!categoryId.value) {
    return items.value;
  }
  return items.value.filter(item => {
    const itemCatId = item.product_category_id ?? item.category_id ?? item.category?.id;
    return itemCatId != null && Number(itemCatId) === Number(categoryId.value);
  });
});

const getCategoryName = (data) => {
  if (data.category?.name) return data.category.name;
  if (data.product_category?.name) return data.product_category.name;
  const catId = data.product_category_id ?? data.category_id ?? data.category?.id;
  if (catId != null) {
    const found = categories.value.find(c => Number(c.id) === Number(catId));
    if (found) return found.name;
  }
  return '-';
};

const onSearch = useDebounceFn(() => {
  setPage(1);
}, 400);

const onCategoryChange = (val) => {
  setCategory(val);
};

const loadCategories = async () => {
  try {
    const res = await getCategories({ limit: 100 });
    categories.value = res.data.data.items || [];
  } catch (error) {
    console.error('Failed to load categories for filter:', error);
  }
};

const confirmDelete = (id) => {
  confirm.require({
    message: "Are u sure u want to delete this product?",
    header: "Confirm Delete",
    icon: "pi pi-exclamation-triangle",
    rejectProps: {
      label: "Cancel",
      severity: "secondary",
      outlined: true
    },
    acceptProps: {
      label: "Delete",
      severity: "danger"
    },
    accept: async () => {
      try {
        await deleteProduct(id);
        toast.add({
          severity: "success",
          summary: "Deleted",
          detail: "Product Removed",
          life: 3000
        });
        fetch();
      } catch (error) {
        toast.add({
          severity: "error",
          summary: "Error",
          detail: "Failed to delete product",
          life: 3000
        });
      }
    }
  });
};

const formatCurrency = (val) => {
  if (val === null || val === undefined) return '-';
  return new Intl.NumberFormat('id-ID', { style: 'currency', currency: 'IDR', minimumFractionDigits: 0 }).format(Number(val));
};

onMounted(() => {
  loadCategories();
  fetch();
});
</script>

<template>
  <div class="min-h-screen bg-surface-50 font-sans text-surface-900">
    <div class="flex justify-between items-center mb-8">
      <div>
        <h1 class="text-2xl font-bold text-surface-900 mb-1">
          Products
        </h1>
        <p class="text-surface-500 text-sm">
          The list here shows all products
        </p>
      </div>

      <Button asChild v-slot="slotProps">
        <RouterLink :to="{ name: 'products-create' }" :class="slotProps?.class">
          Add Product
        </RouterLink>
      </Button>
    </div>

    <div class="bg-white rounded-2xl border border-surface-200 p-2">
      <div class="flex flex-col md:flex-row justify-between items-center px-4 py-4 gap-4">
        <IconField iconPosition="left" class="w-full md:w-80">
          <InputIcon class="pi pi-search text-surface-400" />
          <InputText v-model="search" placeholder="Search" @input="onSearch" />
        </IconField>

        <Select
          :model-value="categoryId"
          :options="categoryOptions"
          optionLabel="name"
          optionValue="id"
          placeholder="Filter by Category"
          filter
          filterPlaceholder="Search category..."
          class="w-full md:w-64"
          @update:model-value="onCategoryChange"
        />
      </div>

      <DataTable :value="displayedItems" :loading="loading" dataKey="id" class="clean-table" :rowHover="true">
        <template #empty>
          <div class="text-center py-8 text-surface-500">
            No products found.
          </div>
        </template>
        <Column field="name" header="Name" class="min-w-[16rem]">
          <template #body="{ data }">
            <div class="flex items-center gap-3">
              <div class="relative">
                <img v-if="data.image" :src="data.image"
                  class="w-10 h-10 rounded-lg object-cover bg-surface-100">
                <div v-else class="w-10 h-10 rounded-lg bg-surface-100 flex items-center justify-center text-surface-400">
                  <i class="pi pi-image"></i>
                </div>
              </div>
              <span class="font-semibold text-surface-900">
                {{ data.name }}
              </span>
            </div>
          </template>
        </Column>

        <Column header="Price">
          <template #body="{ data }">
            <span class="font-medium text-surface-900">{{ formatCurrency(data.price) }}</span>
          </template>
        </Column>

        <Column header="Stock">
          <template #body="{ data }">
            <span class="bg-amber-100 text-amber-800 text-xs px-2.5 py-1 rounded-full font-semibold inline-block">
              {{ data.stock }}
            </span>
          </template>
        </Column>

        <Column header="Category">
          <template #body="{ data }">
            <span class="text-surface-700">{{ getCategoryName(data) }}</span>
          </template>
        </Column>

        <Column header="Actions" style="width: 5rem">
          <template #body="{ data }">
            <div class="flex items-center gap-2">
              <RouterLink :to="{ name: 'products-edit', params: { id: data.id } }">
                <Button icon="pi pi-pencil" text rounded severity="primary"
                  class="w-9! h-9! border-surface-200! text-surface-200! hover:text-primary-600! hover:border-primary-600!" />
              </RouterLink>

              <Button icon="pi pi-trash" text rounded severity="danger"
                class="w-9! h-9! border-surface-200! text-surface-200! hover:text-primary-600"
                @click="confirmDelete(data.id)" />
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

    <ConfirmDialog />
  </div>
</template>
