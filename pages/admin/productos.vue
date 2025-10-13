<template>
  <Toast />
  <div class="p-2 rounded-lg mb-2">
    <p class="text-xl font-bold">Productos</p>
  </div>
  <div>
    <DataTable 
      :value="filteredProductos" tableStyle="min-width: 50rem" 
      show-gridlines size="small"
      paginator :rows="10">
      <template #header>
        <div class="flex justify-between items-center">
          <div>
            <span class="p-input-icon-left">
              <i class="pi pi-search" />
              <InputText v-model="searchQuery" placeholder="Buscar..." />
            </span>
          </div>
          <div class="flex gap-2">
            <Button label="Generar reporte" size="small" @click="reporteProductos" />
            <Button label="Agregar Producto" size="small" @click="AgregarProducto = true"/>
          </div>
        </div>
      </template>
      <template #empty>
        <p class="text-center"> No hay Productos </p>
      </template>
      <template #paginatorcontainer="{ first, last, prevPageCallback, nextPageCallback, totalRecords }">
        <div class="flex items-center gap-2 bg-transparent w-full py-1 px-2 justify-between">
          <Button icon="pi pi-chevron-left" rounded text @click="prevPageCallback" />
          <div class="text-color font-medium">
            <span>Viendo {{ first }} a {{ last }} de {{ totalRecords }} productos</span>
          </div>
          <Button icon="pi pi-chevron-right" rounded text @click="nextPageCallback" />
        </div>
      </template>
      <Column field="id" header="ID" />
      <Column field="nombre" header="Nombre" />
      <Column field="precio" header="Precio">
        <template #body="slotProps">
          {{ slotProps.data.precio }} bs
        </template>
      </Column>
      <Column field="descripcion" header="Descripción" />
      <Column field="categoria" header="Categoria" />
      <Column header="Acciones">
        <template #body="slotProps">
          <div class="flex items-center justify-center">
            <Button label="Editar" variant="text" @click="idProducto = slotProps.data.id, ModificarProducto = true"/>
          </div>
        </template>
      </Column>
    </DataTable>
  </div>

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
import modalAgregarProducto from '~/components/admin/productos/modalAgregarProducto.vue'
import modalModificarProducto from '~/components/admin/productos/modalModificarProducto.vue'
import { server } from '~/server/server'

definePageMeta({ layout : 'menu-admin' })

const toast = useToast()
const Productos = ref<any[]>([])
const AgregarProducto = ref(false)
const ModificarProducto = ref(false)
const idProducto = ref(0)
const searchQuery = ref('')

const filteredProductos = computed(() => {
  if (!searchQuery.value) {
    return Productos.value
  }
  const query = searchQuery.value.toLowerCase()
  return Productos.value.filter(p => 
    p.id.toString().includes(query) || 
    p.nombre.toLowerCase().includes(query)
  )
})

onMounted( async () => {
  await obtenerProductos()
})

async function obtenerProductos() {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/productos', {
      method: 'GET'
    })
    res.sort((a, b) => a.categoria.localeCompare(b.categoria));
    Productos.value = res
  } catch (err) {
    console.error(err)
  }
}

async function reporteProductos() {
  try {
    const reporte = await fetch(server.HOST + '/api/v1/reportes/productos', {
      method: 'GET'
    })
    const blob = await reporte.blob()
    const url = URL.createObjectURL(blob)
    window.open(url, '_blank')
  } catch(err) {
    console.error(err)
  }
}
</script>