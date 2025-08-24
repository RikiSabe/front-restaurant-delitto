<template>
  <Toast />
  <div class="p-2 rounded-lg mb-2">
    <p class="text-xl font-bold">Categorias de los Insumos</p>
  </div>
  <div>
    <DataTable 
      :value="Categorias" tableStyle="min-width: 50rem" 
      show-gridlines size="small"
      paginator :rows="5">
      <template #header>
        <div class="flex justify-end">
          <Button label="Agregar Categoria" size="small" @click="AgregarCategoria = true"/>
        </div>
      </template>
      <template #empty>
        <p class="text-center"> No hay Categorias </p>
      </template>
      <template #paginatorcontainer="{ first, last, prevPageCallback, nextPageCallback, totalRecords }">
        <div class="flex items-center gap-2 bg-transparent w-full py-1 px-2 justify-between">
          <Button icon="pi pi-chevron-left" rounded text @click="prevPageCallback" />
          <div class="text-color font-medium">
            <span>Viendo {{ first }} a {{ last }} de {{ totalRecords }} categorias </span>
          </div>
          <Button icon="pi pi-chevron-right" rounded text @click="nextPageCallback" />
        </div>
      </template>
      <Column field="id" header="ID" />
      <Column field="nombre" header="Nombre" />
      <Column field="descripcion" header="Descripción" />
      <Column field="estado" header="Estado" />
      <Column header="Acciones">
        <template #body="slotProps">
          <div class="flex items-center justify-center">
            <Button label="Editar" variant="text" @click="idCategoria = slotProps.data.id, ModificarCategoria = true"/>
          </div>
        </template>
      </Column>
    </DataTable>
  </div>

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
import modalAgregarCategoria from '~/components/admin/categorias-insumos/modalAgregarCategoria.vue'
import modalModificarCategoria from '~/components/admin/categorias-insumos/modalModificarCategoria.vue'

import { server } from '~/server/server'

definePageMeta({ layout : 'menu-admin' })

const toast = useToast()
const Categorias = ref<any[]>([])
const AgregarCategoria = ref(false)
const ModificarCategoria = ref(false)
const idCategoria = ref(0)

onMounted( async () => {
  await obtenerCategorias()
})

async function obtenerCategorias() {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/categorias-insumos', {
      method: 'GET'
    })
    Categorias.value = res
  } catch (err) {
    console.error(err)
  }
}
</script>