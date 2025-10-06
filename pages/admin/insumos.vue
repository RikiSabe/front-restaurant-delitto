<template>
  <Toast />
  <div class="p-2 rounded-lg mb-2">
    <p class="text-xl font-bold">Insumos</p>
  </div>
  <div>
    <DataTable 
      :value="Insumos" tableStyle="min-width: 50rem" 
      show-gridlines size="small"
      paginator :rows="5">
      <template #header>
        <div class="flex justify-between">
          <Button label="Generar reporte" size="small" @click="reporteInsumos" />
          <Button label="Agregar Insumo" size="small" @click="AgregarInsumo = true"/>
        </div>
      </template>
      <template #empty>
        <p class="text-center"> No hay Insumos </p>
      </template>
      <template #paginatorcontainer="{ first, last, prevPageCallback, nextPageCallback, totalRecords }">
        <div class="flex items-center gap-2 bg-transparent w-full py-1 px-2 justify-between">
          <Button icon="pi pi-chevron-left" rounded text @click="prevPageCallback" />
          <div class="text-color font-medium">
            <span>Viendo {{ first }} a {{ last }} de {{ totalRecords }} insumos </span>
          </div>
          <Button icon="pi pi-chevron-right" rounded text @click="nextPageCallback" />
        </div>
      </template>
      <Column field="id" header="ID" />
      <Column field="nombre" header="Nombre" />
      <Column field="nombre_proveedor" header="Proveedor" />
      <Column field="stock_actual" header="Stock Actual" />
      <Column field="stock_minimo" header="Stock Minimo" />
      <Column field="unidad_medida" header="Unidad de Medida" />
      <Column header="Acciones">
        <template #body="slotProps">
          <div class="flex items-center justify-center">
            <Button label="Editar" variant="text" @click="idInsumo = slotProps.data.id, ModificarInsumo = true"/>
          </div>
        </template>
      </Column>
    </DataTable>
  </div>

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
import modalAgregarInsumo from '~/components/admin/insumos/modalAgregarInsumo.vue'
import modalModificarInsumo from '~/components/admin/insumos/modalModificarInsumo.vue'
import { server } from '~/server/server'

definePageMeta({ layout : 'menu-admin' })

const toast = useToast()
const Insumos = ref<any[]>([])
const AgregarInsumo = ref(false), ModificarInsumo = ref(false)
const idInsumo = ref(0)

onMounted( async () => {
  await obtenerInsumos()
})

async function obtenerInsumos() {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/insumos', {
      method: 'GET'
    })
    Insumos.value = res
  } catch (err) {
    console.error(err)
  }
}

async function reporteInsumos() {
  try {
    const reporte = await fetch(server.HOST + '/api/v1/reportes/insumos', {
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