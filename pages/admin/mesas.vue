<template>
  <Toast />
  <div class="p-2 rounded-lg mb-2">
    <p class="text-xl font-bold">Mesas</p>
  </div>
  <div>
    <DataTable
      :value="filteredMesas" tableStyle="min-width: 50rem"
      show-gridlines size="small"
      paginator :rows="5">
      <template #header>
        <div class="flex justify-between items-center">
          <div>
            <span class="p-input-icon-left">
              <i class="pi pi-search" />
              <InputText v-model="searchQuery" placeholder="Buscar..." />
            </span>
          </div>
          <Button label="Agregar Mesa" size="small" @click="AgregarMesa = true"/>
        </div>
      </template>
      <template #empty>
        <p class="text-center"> No hay Mesas registradas </p>
      </template>
      <template #paginatorcontainer="{ first, last, prevPageCallback, nextPageCallback, totalRecords }">
        <div class="flex items-center gap-2 bg-transparent w-full py-1 px-2 justify-between">
          <Button icon="pi pi-chevron-left" rounded text @click="prevPageCallback" />
          <div class="text-color font-medium">
            <span>Viendo {{ first }} a {{ last }} de {{ totalRecords }} mesas </span>
          </div>
          <Button icon="pi pi-chevron-right" rounded text @click="nextPageCallback" />
        </div>
      </template>
      <Column field="id" header="#" />
      <Column field="nombre" header="Nombre" />
      <Column field="capacidad" header="Capacidad" />
      <Column field="estado" header="Estado" />
      <Column header="Acciones">
        <template #body="slotProps">
          <div class="flex items-center justify-center">
            <Button label="Editar" variant="text" @click="idMesa = slotProps.data.id, ModificarMesa = true"/>
          </div>
        </template>
      </Column>
    </DataTable>
  </div>
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
import modalAgregarMesa from '~/components/admin/mesas/modalAgregarMesa.vue'
import modalModificarMesa from '~/components/admin/mesas/modalModificarMesa.vue'
import { server } from '~/server/server'

definePageMeta({ layout : 'menu-admin' })

const toast = useToast()
const Mesas = ref<any[]>([])
const AgregarMesa = ref(false), ModificarMesa = ref(false)
const idMesa = ref(0)
const searchQuery = ref('')

const filteredMesas = computed(() => {
  if (!searchQuery.value) {
    return Mesas.value
  }
  const query = searchQuery.value.toLowerCase()
  return Mesas.value.filter(m =>
    m.id.toString().includes(query) ||
    m.nombre.toLowerCase().includes(query)
  )
})

onMounted( async () => {
  await ObtenerMesas()
})

const ObtenerMesas = async () => {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/mesas', {
      method: 'GET'
    })
    Mesas.value = res
  } catch (err) {
    console.error(err)
  }
}

</script>