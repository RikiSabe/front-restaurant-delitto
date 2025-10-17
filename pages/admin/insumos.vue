<template>
  <Toast />

  <!-- Title -->
  <h2 class="text-2xl font-bold mb-4">Insumos</h2>

  <!-- Controls Bar -->
  <div class="flex justify-between items-center mb-4">
    <div class="flex gap-2">
      <Button label="Nuevo Insumo" icon="pi pi-plus" size="small" @click="AgregarInsumo = true"/>
      <Button label="Generar Reporte" icon="pi pi-file-pdf" size="small" severity="secondary" @click="reporteInsumos" />
    </div>
    <span class="p-input-icon-left">
      <i class="pi pi-search" />
      <InputText v-model="searchQuery" placeholder="Buscar..." />
    </span>
  </div>

  <!-- Data Table -->
  <div>
    <DataTable 
      :value="filteredInsumos" 
      tableStyle="min-width: 50rem" 
      size="small"
      stripedRows
      removableSort
      paginator :rows="10"
      :rowsPerPageOptions="[10, 20, 50]"
      paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink RowsPerPageDropdown"
      currentPageReportTemplate="Mostrando {first} a {last} de {totalRecords} insumos"
      :pt="{ thead: { class: 'bg-amber-300' } }"
    >
      <template #empty>
        <p class="text-center p-4">No hay insumos para mostrar.</p>
      </template>
      
      <Column field="id" header="ID" sortable />
      <Column field="nombre" header="Nombre" sortable />
      <Column field="nombre_proveedor" header="Proveedor" sortable />
      <Column field="stock_actual" header="Stock Actual" sortable />
      <Column field="stock_minimo" header="Stock Mínimo" />
      <Column field="unidad_medida" header="Unidad" />
      <Column header="Acciones" style="width: 10rem">
        <template #body="slotProps">
          <div class="flex items-center justify-center">
            <Button label="Editar" variant="text" size="small" @click="idInsumo = slotProps.data.id; ModificarInsumo = true"/>
          </div>
        </template>
      </Column>
    </DataTable>
  </div>

  <!-- Modals -->
  <modalAgregarInsumo 
    :open="AgregarInsumo"
    v-if="AgregarInsumo"
    @close="AgregarInsumo = false"
    @update="obtenerInsumos"
    @success="toast.add({ severity: 'success', summary: 'Agregado Exitoso', life: 3000 })"
    @error="toast.add({ severity: 'error', summary: 'Error al Agregar', life: 3000 })" />

  <modalModificarInsumo 
    :open="ModificarInsumo"
    :id="idInsumo"
    v-if="ModificarInsumo"
    @close="ModificarInsumo = false"
    @update="obtenerInsumos"
    @success="toast.add({ severity: 'success', summary: 'Modificado Exitoso', life: 3000 })"
    @error="toast.add({ severity: 'error', summary: 'Error al Modificar', life: 3000 })" />
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import modalAgregarInsumo from '~/components/admin/insumos/modalAgregarInsumo.vue';
import modalModificarInsumo from '~/components/admin/insumos/modalModificarInsumo.vue';
import { server } from '~/server/server';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Button from 'primevue/button';
import InputText from 'primevue/inputtext';
import Toast from 'primevue/toast';
import { useToast } from 'primevue/usetoast';

definePageMeta({ layout : 'menu-admin' });

const toast = useToast();
const Insumos = ref<any[]>([]);
const AgregarInsumo = ref(false), ModificarInsumo = ref(false);
const idInsumo = ref(0);
const searchQuery = ref('');

const filteredInsumos = computed(() => {
  if (!searchQuery.value) {
    return Insumos.value;
  }
  const query = searchQuery.value.toLowerCase();
  return Insumos.value.filter(i =>
    (i.id?.toString() || '').includes(query) ||
    (i.nombre?.toLowerCase() || '').includes(query) ||
    (i.nombre_proveedor?.toLowerCase() || '').includes(query)
  );
});

onMounted(async () => {
  await obtenerInsumos();
});

async function obtenerInsumos() {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/insumos', {
      method: 'GET'
    });
    Insumos.value = res;
  } catch (err) {
    console.error(err);
  }
}

async function reporteInsumos() {
  try {
    const reporte = await fetch(server.HOST + '/api/v1/reportes/insumos', {
      method: 'GET'
    });
    const blob = await reporte.blob();
    const url = URL.createObjectURL(blob);
    window.open(url, '_blank');
  } catch(err) {
    console.error(err);
  }
}
</script>
