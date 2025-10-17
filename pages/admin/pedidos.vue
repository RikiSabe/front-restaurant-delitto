<template>
  <Toast />

  <!-- Title -->
  <h2 class="text-2xl font-bold mb-4">Historial de Pedidos</h2>

  <!-- Controls Bar -->
  <div class="flex justify-between items-center mb-4">
    <div class="flex gap-2">
      <Button label="Expandir Todo" icon="pi pi-plus" size="small" severity="secondary" @click="expandAll" />
      <Button label="Colapsar Todo" icon="pi pi-minus" size="small" severity="secondary" @click="collapseAll" />
      <Button label="Reporte Diario" icon="pi pi-file-pdf" size="small" severity="secondary" @click="reporteDiario" />
      <Button label="Ver por ID" icon="pi pi-search-plus" size="small" severity="secondary" @click="pedidoID = true" />
    </div>
    <span class="p-input-icon-left">
      <i class="pi pi-search" />
      <InputText v-model="searchQuery" placeholder="Buscar por ID, estado, origen..." />
    </span>
  </div>

  <!-- Data Table -->
  <div>
    <DataTable 
      v-model:expandedRows="expandedRows"
      :value="filteredPedidos" 
      dataKey="id"
      tableStyle="min-width: 50rem" 
      size="small"
      stripedRows
      removableSort
      paginator :rows="10"
      :rowsPerPageOptions="[10, 20, 50]"
      paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink RowsPerPageDropdown"
      currentPageReportTemplate="Mostrando {first} a {last} de {totalRecords} pedidos"
      :pt="{ thead: { class: 'bg-amber-300' } }"
    >
      <template #empty>
        <p class="text-center p-4">No hay pedidos para mostrar.</p>
      </template>

      <Column expander style="width: 4rem" />
      <Column field="id" header="ID" sortable />
      <Column field="estado" header="Estado" sortable />
      <Column field="origen" header="Origen del Pedido" sortable />
      <Column field="fecha" header="Fecha" sortable >
        <template #body="slotProps">
          {{ formatearFecha(slotProps.data.fecha) }}
        </template>
      </Column>

      <template #expansion="slotProps">
        <div class="p-4 bg-gray-50">
          <h5 class="mb-2 font-bold">Productos del Pedido #{{ slotProps.data.id }}</h5>
          <DataTable :value="slotProps.data.productos" size="small" showGridlines>
            <Column field="nombre" header="Nombre" />
            <Column field="categoria" header="Categoría" />
            <Column field="precio" header="Precio" />
            <Column field="cantidad" header="Cantidad" />
            <Column field="subtotal" header="Subtotal" />
            <template #footer>
              <div class="flex justify-end font-bold pr-4">
                Total: {{ calcularTotalPedido(slotProps.data.productos).toFixed(2) }} Bs.
              </div>
            </template>
          </DataTable>
        </div>
      </template>
    </DataTable>
  </div>

  <!-- Modals -->
  <Dialog v-model:visible="pedidoID" header="Ver Reporte de Pedido Individual" :style="{ width: '25rem' }">
    <div class="flex flex-col gap-4 p-4">
      <p class="text-center">Ingrese el ID del pedido para generar el reporte.</p>
      <InputNumber v-model="ID" placeholder="ID del pedido" fluid />
      <Button label="Generar Reporte" icon="pi pi-file-pdf" @click="reporteIndividual" />
    </div>
  </Dialog>
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import { server } from '~/server/server';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Button from 'primevue/button';
import InputText from 'primevue/inputtext';
import Dialog from 'primevue/dialog';
import InputNumber from 'primevue/inputnumber';
import Toast from 'primevue/toast';
import { useToast } from 'primevue/usetoast';

definePageMeta({ layout: 'menu-admin' });

const toast = useToast();
const Pedidos = ref<any[]>([]);
const expandedRows = ref<any>(null);
const pedidoID = ref(false);
const ID = ref<number>();
const searchQuery = ref('');

const filteredPedidos = computed(() => {
  if (!searchQuery.value) {
    return Pedidos.value;
  }
  const query = searchQuery.value.toLowerCase();
  return Pedidos.value.filter(p =>
    (p.id?.toString() || '').includes(query) ||
    (p.estado?.toLowerCase() || '').includes(query) ||
    (p.origen?.toLowerCase() || '').includes(query) ||
    formatearFecha(p.fecha).includes(query)
  );
});

onMounted(async () => {
  await obtenerPedidos();
});

async function obtenerPedidos() {
  try {
    const res: any[] = await $fetch(server.HOST + '/api/v1/pedidos', {
      method: 'GET'
    });
    Pedidos.value = res;
  } catch (err) {
    console.error(err);
  }
}

function calcularTotalPedido(productos: any[]) {
  if (!productos || productos.length === 0) {
    return 0;
  }
  return productos.reduce((total, producto) => total + producto.subtotal, 0);
}

function expandAll() {
  expandedRows.value = Pedidos.value.reduce((acc: any, p: any) => {
    acc[p.id] = true;
    return acc;
  }, {});
}

function collapseAll() {
  expandedRows.value = null;
}

function formatearFecha(fechaISO: string): string {
  if (!fechaISO) return '';
  const date = new Date(fechaISO);
  const dia = String(date.getDate()).padStart(2, '0');
  const mes = String(date.getMonth() + 1).padStart(2, '0');
  const anio = date.getFullYear();
  return `${dia}/${mes}/${anio}`;
}

async function reporteDiario() {
  try {
    const reporte = await fetch(server.HOST + '/api/v1/reportes/diario', {
      method: 'GET'
    });
    const blob = await reporte.blob();
    const url = URL.createObjectURL(blob);
    window.open(url, '_blank');
  } catch(err) {
    console.error(err);
  }
}

async function reporteIndividual() {
  if (!ID.value) {
    toast.add({ severity: 'warn', summary: 'Entrada Inválida', detail: 'Por favor, ingrese un ID de pedido.', life: 3000 });
    return;
  }
  try {
    const reporte = await fetch(`${server.HOST}/api/v1/reportes/individual/${ID.value}`, {
      method: 'GET'
    });
    if (reporte.ok) {
      const blob = await reporte.blob();
      const url = URL.createObjectURL(blob);
      window.open(url, '_blank');
      pedidoID.value = false;
    } else {
      toast.add({ severity: 'error', summary: 'Error', detail: 'No se pudo generar el reporte para el ID proporcionado.', life: 3000 });
    }
  } catch(err) {
    console.error(err);
    toast.add({ severity: 'error', summary: 'Error de Red', detail: 'No se pudo conectar al servidor.', life: 3000 });
  }
}
</script>
