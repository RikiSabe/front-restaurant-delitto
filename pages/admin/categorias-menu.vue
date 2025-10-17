<template>
  <Toast />

  <!-- Title -->
  <h2 class="text-2xl font-bold mb-4">Categorías de Menú</h2>

  <!-- Controls Bar -->
  <div class="flex justify-between items-center mb-4">
    <Button label="Nueva Categoría" icon="pi pi-plus" size="small" @click="AgregarCategoria = true"/>
    <span class="p-input-icon-left">
      <i class="pi pi-search" />
      <InputText v-model="searchQuery" placeholder="Buscar..." />
    </span>
  </div>

  <!-- Data Table -->
  <div>
    <DataTable 
      :value="filteredCategorias" 
      tableStyle="min-width: 50rem" 
      size="small"
      stripedRows
      removableSort
      paginator :rows="10"
      :rowsPerPageOptions="[10, 20, 50]"
      paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink RowsPerPageDropdown"
      currentPageReportTemplate="Mostrando {first} a {last} de {totalRecords} categorías"
      :pt="{ thead: { class: 'bg-amber-300' } }"
    >
      <template #empty>
        <p class="text-center p-4">No hay categorías para mostrar.</p>
      </template>
      
      <Column field="id" header="ID" sortable />
      <Column field="nombre" header="Nombre" sortable />
      <Column field="descripcion" header="Descripción" />
      <Column field="estado" header="Estado" sortable />
      <Column header="Acciones" style="width: 10rem">
        <template #body="slotProps">
          <div class="flex items-center justify-center">
            <Button label="Editar" variant="text" size="small" @click="idCategoria = slotProps.data.id; ModificarCategoria = true"/>
          </div>
        </template>
      </Column>
    </DataTable>
  </div>

  <!-- Modals -->
  <modalAgregarCategoria 
    :open="AgregarCategoria"
    v-if="AgregarCategoria"
    @close="AgregarCategoria = false"
    @update="obtenerCategorias"
    @success="toast.add({ severity: 'success', summary: 'Agregado Exitoso', life: 3000 })"
    @error="toast.add({ severity: 'error', summary: 'Error al Agregar', life: 3000 })" />

  <modalModificarCategoria 
    :open="ModificarCategoria"
    :id="idCategoria"
    v-if="ModificarCategoria"
    @close="ModificarCategoria = false"
    @update="obtenerCategorias"
    @success="toast.add({ severity: 'success', summary: 'Modificado Exitoso', life: 3000 })"
    @error="toast.add({ severity: 'error', summary: 'Error al Modificar', life: 3000 })" />
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import modalAgregarCategoria from '~/components/admin/categorias-menu/modalAgregarCategoria.vue';
import modalModificarCategoria from '~/components/admin/categorias-menu/modalModificarCategoria.vue';
import { server } from '~/server/server';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Button from 'primevue/button';
import InputText from 'primevue/inputtext';
import Toast from 'primevue/toast';
import { useToast } from 'primevue/usetoast';

definePageMeta({ layout : 'menu-admin' });

const toast = useToast();
const Categorias = ref<any[]>([]);
const AgregarCategoria = ref(false);
const ModificarCategoria = ref(false);
const idCategoria = ref(0);
const searchQuery = ref('');

const filteredCategorias = computed(() => {
  if (!searchQuery.value) {
    return Categorias.value;
  }
  const query = searchQuery.value.toLowerCase();
  return Categorias.value.filter(cat =>
    (cat.id?.toString() || '').includes(query) ||
    (cat.nombre?.toLowerCase() || '').includes(query) ||
    (cat.descripcion?.toLowerCase() || '').includes(query) ||
    (cat.estado?.toLowerCase() || '').includes(query)
  );
});

onMounted(async () => {
  await obtenerCategorias();
});

async function obtenerCategorias() {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/categorias-productos', {
      method: 'GET'
    });
    Categorias.value = res;
  } catch (err) {
    console.error(err);
  }
}
</script>
