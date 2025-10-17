<template>
  <Toast />

  <!-- Title -->
  <h2 class="text-2xl font-bold mb-4">Productos</h2>

  <!-- Controls Bar -->
  <div class="flex justify-between items-center mb-4">
    <div class="flex gap-2">
      <Button label="Nuevo" icon="pi pi-plus" size="small" @click="AgregarProducto = true"/>
      <Button label="Generar Reporte" icon="pi pi-file-pdf" size="small" severity="secondary" @click="reporteProductos" />
    </div>
    <span class="p-input-icon-left">
      <i class="pi pi-search" />
      <InputText v-model="searchQuery" placeholder="Buscar..." />
    </span>
  </div>

  <!-- Data Table -->
  <div>
    <DataTable 
      :value="filteredProductos" 
      tableStyle="min-width: 50rem" 
      size="small"
      stripedRows
      removableSort
      paginator :rows="10"
      :rowsPerPageOptions="[10, 20, 50]"
      paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink RowsPerPageDropdown"
      currentPageReportTemplate="Mostrando {first} a {last} de {totalRecords} productos"
      :pt="{ thead: { class: 'bg-amber-300' } }"
    >
      <template #empty>
        <p class="text-center p-4">No hay productos para mostrar.</p>
      </template>
      
      <Column field="id" header="ID" sortable />
      <Column field="nombre" header="Nombre" sortable />
      <Column field="precio" header="Precio" sortable >
        <template #body="slotProps">
          {{ slotProps.data.precio }} Bs.
        </template>
      </Column>
      <Column field="descripcion" header="Descripción" />
      <Column field="categoria" header="Categoría" sortable />
      <Column header="Acciones" style="width: 10rem">
        <template #body="slotProps">
          <div class="flex items-center justify-center">
            <Button label="Editar" variant="text" size="small" @click="idProducto = slotProps.data.id; ModificarProducto = true"/>
          </div>
        </template>
      </Column>
    </DataTable>
  </div>

  <!-- Modals -->
  <modalAgregarProducto 
    :open="AgregarProducto"
    v-if="AgregarProducto"
    @close="AgregarProducto = false"
    @update="obtenerProductos"
    @success="toast.add({ severity: 'success', summary: 'Agregado Exitoso', life: 3000 })"
    @error="toast.add({ severity: 'error', summary: 'Error al Agregar', life: 3000 })" />

  <modalModificarProducto 
    :open="ModificarProducto"
    :id="idProducto"
    v-if="ModificarProducto"
    @close="ModificarProducto = false"
    @update="obtenerProductos"
    @success="toast.add({ severity: 'success', summary: 'Modificado Exitoso', life: 3000 })"
    @error="toast.add({ severity: 'error', summary: 'Error al Modificar', life: 3000 })" />
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import modalAgregarProducto from '~/components/admin/productos/modalAgregarProducto.vue';
import modalModificarProducto from '~/components/admin/productos/modalModificarProducto.vue';
import { server } from '~/server/server';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Button from 'primevue/button';
import InputText from 'primevue/inputtext';
import Toast from 'primevue/toast';
import { useToast } from 'primevue/usetoast';

definePageMeta({ layout : 'menu-admin' });

const toast = useToast();
const Productos = ref<any[]>([]);
const AgregarProducto = ref(false);
const ModificarProducto = ref(false);
const idProducto = ref(0);
const searchQuery = ref('');

const filteredProductos = computed(() => {
  if (!searchQuery.value) {
    return Productos.value;
  }
  const query = searchQuery.value.toLowerCase();
  return Productos.value.filter(p => 
    (p.id?.toString() || '').includes(query) || 
    (p.nombre?.toLowerCase() || '').includes(query) ||
    (p.categoria?.toLowerCase() || '').includes(query)
  );
});

onMounted(async () => {
  await obtenerProductos();
});

async function obtenerProductos() {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/productos', {
      method: 'GET'
    });
    res.sort((a, b) => a.categoria.localeCompare(b.categoria));
    Productos.value = res;
  } catch (err) {
    console.error(err);
  }
}

async function reporteProductos() {
  try {
    const reporte = await fetch(server.HOST + '/api/v1/reportes/productos', {
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
