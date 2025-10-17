<template>
  <Toast />

  <!-- Title -->
  <h2 class="text-2xl font-bold mb-4">Mesas</h2>

  <!-- Controls Bar -->
  <div class="flex justify-between items-center mb-4">
    <Button label="Nueva Mesa" icon="pi pi-plus" size="small" @click="AgregarMesa = true"/>
    <span class="p-input-icon-left">
      <i class="pi pi-search" />
      <InputText v-model="searchQuery" placeholder="Buscar..." />
    </span>
  </div>

  <!-- Data Table -->
  <div>
    <DataTable 
      :value="filteredMesas" 
      tableStyle="min-width: 50rem" 
      size="small"
      stripedRows
      removableSort
      paginator :rows="10"
      :rowsPerPageOptions="[5, 10, 25, 50]"
      paginatorTemplate="FirstPageLink PrevPageLink PageLinks NextPageLink LastPageLink RowsPerPageDropdown"
      currentPageReportTemplate="Mostrando {first} a {last} de {totalRecords} mesas"
      :pt="{ thead: { class: 'bg-amber-300' } }"
    >
      <template #empty>
        <p class="text-center p-4">No hay mesas para mostrar.</p>
      </template>
      
      <Column field="id" header="#" sortable />
      <Column field="nombre" header="Nombre" sortable />
      <Column field="capacidad" header="Capacidad" sortable />
      <Column field="estado" header="Estado" sortable />
      <Column header="Acciones" style="width: 10rem">
        <template #body="slotProps">
          <div class="flex items-center justify-center">
            <Button label="Editar" variant="text" size="small" @click="idMesa = slotProps.data.id; ModificarMesa = true"/>
          </div>
        </template>
      </Column>
    </DataTable>
  </div>

  <!-- Modals -->
  <modalAgregarMesa
    :open="AgregarMesa"
    v-if="AgregarMesa"
    @close="AgregarMesa = false" 
    @update="ObtenerMesas"
    @success="toast.add({ severity: 'success', summary: 'Mesa Agregada Exitosamente', life: 3000 })" 
    @fail="toast.add({ severity: 'error', summary: 'Error al agregar Mesa', life: 3000 })"/>
  
  <modalModificarMesa 
    :open="ModificarMesa"
    :id="idMesa"
    v-if="ModificarMesa"
    @close="ModificarMesa = false"
    @update="ObtenerMesas"
    @success="toast.add({ severity: 'success', summary: 'Mesa Modificada Existosamente', life:3000 })"
    @fail="toast.add({ severity: 'error', summary: 'Error al actualizar Mesa', life: 3000 })" />
</template>

<script setup lang="ts">
import { ref, computed, onMounted } from 'vue';
import modalAgregarMesa from '~/components/admin/mesas/modalAgregarMesa.vue';
import modalModificarMesa from '~/components/admin/mesas/modalModificarMesa.vue';
import { server } from '~/server/server';
import DataTable from 'primevue/datatable';
import Column from 'primevue/column';
import Button from 'primevue/button';
import InputText from 'primevue/inputtext';
import Toast from 'primevue/toast';
import { useToast } from 'primevue/usetoast';

definePageMeta({ layout : 'menu-admin' });

const toast = useToast();
const Mesas = ref<any[]>([]);
const AgregarMesa = ref(false), ModificarMesa = ref(false);
const idMesa = ref(0);
const searchQuery = ref('');

const filteredMesas = computed(() => {
  if (!searchQuery.value) {
    return Mesas.value;
  }
  const query = searchQuery.value.toLowerCase();
  return Mesas.value.filter(m =>
    (m.id?.toString() || '').includes(query) ||
    (m.nombre?.toLowerCase() || '').includes(query) ||
    (m.estado?.toLowerCase() || '').includes(query)
  );
});

onMounted(async () => {
  await ObtenerMesas();
});

const ObtenerMesas = async () => {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/mesas', {
      method: 'GET'
    });
    Mesas.value = res;
  } catch (err) {
    console.error(err);
  }
}
</script>
