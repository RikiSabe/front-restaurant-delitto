<template>
  <div class="card">
    
    <div class="p-2 rounded-lg mb-2">
      <p class="text-xl font-bold">Historial de Pedidos</p>
    </div>

    <DataTable
      v-model:expandedRows="expandedRows" :value="Pedidos"
      dataKey="id" showGridlines size="small"
      paginator :rows="5" tableStyle="min-width: 70rem">
      
      <template #header>
        <div class="flex flex-wrap justify-end gap-2">
          <Button text label="Expandir Todo" @click="expandAll" size="small" />
          <Button text label="Colapsar Todo" @click="collapseAll" size="small" />
        </div>
      </template>

      <Column expander style="width: 4rem" />
      <Column field="id" header="ID" />
      <Column field="estado" header="Estado" />
      <Column field="fecha" header="Fecha">
        <template #body="slotProps">
          {{ formatearFecha(slotProps.data.fecha) }}
        </template>
      </Column>
      <Column field="id_mesa" header="# Mesa" />
      <Column header="Capacidad">
        <template #body="slotProps">
          {{ slotProps.data.mesa?.capacidad ?? 'N/A' }}
        </template>
      </Column>

      <template #expansion="slotProps">
        <div class="p-4 flex flex-col gap-4">
          <div>
            <h5 class="mb-2 font-bold">Usuarios</h5>
            <DataTable :value="slotProps.data.usuarios" size="small" showGridlines>
              <Column field="id" header="ID" />
              <Column field="nombre" header="Nombre" />
              <Column field="rol" header="Rol" />
            </DataTable>
          </div>

          <div>
            <h5 class="mb-2 font-bold">Productos</h5>
            <DataTable :value="slotProps.data.productos" size="small" showGridlines>
              <Column field="id" header="ID" />
              <Column field="nombre" header="Nombre" />
              <Column field="precio" header="Precio" />
            </DataTable>
          </div>

          <div>
            <h5 class="mb-2 font-bold">Insumos</h5>
            <DataTable :value="slotProps.data.insumos" size="small" showGridlines>
              <Column field="id" header="ID" />
              <Column field="nombre" header="Nombre" />
              <Column field="cantidad" header="Cantidad" />
            </DataTable>
          </div>
        </div>
      </template>

      <template #empty>
        <p class="text-center"> No hay pedidos </p>
      </template>

      <template #paginatorcontainer="{ first, last, prevPageCallback, nextPageCallback, totalRecords }">
        <div class="flex items-center gap-2 bg-transparent w-full py-1 px-2 justify-between">
          <Button icon="pi pi-chevron-left" rounded text @click="prevPageCallback" />
          <div class="text-color font-medium">
            <span>Viendo {{ first }} a {{ last }} de {{ totalRecords }} pedidos</span>
          </div>
          <Button icon="pi pi-chevron-right" rounded text @click="nextPageCallback" />
        </div>
      </template>
    </DataTable>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { useToast } from 'primevue/usetoast'
import { server } from '~/server/server'

definePageMeta({ layout: 'menu-cajero' })

const toast = useToast()
const Pedidos = ref<any[]>([])
const expandedRows = ref<any>(null)

onMounted(async () => {
  await obtenerPedidos()
})

async function obtenerPedidos() {
  try {
    const res: any[] = await $fetch(server.HOST + '/api/v1/pedidos', {
      method: 'GET'
    })
    Pedidos.value = res
  } catch (err) {
    console.error(err)
  }
}

function expandAll() {
  expandedRows.value = Pedidos.value.reduce((acc: any, p: any) => {
    acc[p.id] = true
    return acc
  }, {})
}

function collapseAll() {
  expandedRows.value = null
}

function formatearFecha(fechaISO: string): string {
  const date = new Date(fechaISO)
  const dia = String(date.getDate()).padStart(2, '0')
  const mes = String(date.getMonth() + 1).padStart(2, '0')
  const anio = date.getFullYear()
  return `${dia}/${mes}/${anio}`
}

</script>
