<template>
  <Toast />
  <div class="card">
    <div class="p-2 rounded-lg mb-2">
      <p class="text-xl font-bold">Historial de Pedidos</p>
    </div>
    <DataTable
      v-model:expandedRows="expandedRows" :value="Pedidos"
      dataKey="id" showGridlines size="small"
      paginator :rows="5" tableStyle="min-width: 50rem">
      <template #header>
        <div class="flex flex-wrap justify-end gap-2">
          <Button text label="Expandir Todo" @click="expandAll" size="small" />
          <Button text label="Colapsar Todo" @click="collapseAll" size="small" />
        </div>
      </template>
      <Column expander style="width: 4rem" />
      <Column field="id" header="ID" />
      <Column field="estado" header="Estado" />
      <Column field="origen" header="Origen del Pedido" />
      <Column field="fecha" header="Fecha">
        <template #body="slotProps">
          {{ formatearFecha(slotProps.data.fecha) }}
        </template>
      </Column>
      <template #expansion="slotProps">
        <div class="p-4 flex flex-col gap-4">
          <div>
            <h5 class="mb-2 font-bold">Productos</h5>
            <DataTable :value="slotProps.data.productos" size="small" showGridlines>
              <Column field="nombre" header="Nombre" />
              <Column field="categoria" header="Categoría" />
              <Column field="precio" header="Precio" />
              <Column field="cantidad" header="#" />
              <Column field="subtotal" header="SubTotal" />
              <template #footer>
                <div class="flex justify-end font-bold pr-4">
                  Total: {{ calcularTotalPedido(slotProps.data.productos).toFixed(2) }} bs
                </div>
              </template>
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

  <section class="grid grid-cols-2 gap-2 mt-2">
    <Button label="Generar reporte diario" fluid variant="outlined" @click="reporteDiario" />
    <Button label="Ver pedido por ID" fluid variant="outlined" @click="pedidoID = true" />
  </section>

  <Dialog v-model:visible="pedidoID" header=" " :style="{ width: '25rem' }">
    <div class="flex flex-col gap-4">
      <p class="text-center"> Ingrese el ID del pedido para buscar </p>
      <InputNumber v-model="ID" placeholder="ID del pedido" fluid />
      <Button label="Ver pedido" fluid variant="outlined" @click="reporteIndividual" />
    </div>
  </Dialog>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue'
import { server } from '~/server/server'

definePageMeta({ layout: 'menu-admin' })

const toast = useToast()
const Pedidos = ref<any[]>([])
const expandedRows = ref<any>(null)
const pedidoID = ref(false)
const ID = ref()

onMounted(async () => {
  await obtenerPedidos()
})

async function obtenerPedidos() {
  try {
    const res: any[] = await $fetch(server.HOST + '/api/v1/pedidos', {
      method: 'GET'
    })
    Pedidos.value = res
    console.log(JSON.stringify(Pedidos.value, null, 2))
  } catch (err) {
    console.error(err)
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

async function reporteDiario() {
  try {
    const reporte = await fetch(server.HOST + '/api/v1/reportes/diario', {
      method: 'GET'
    })
    const blob = await reporte.blob()
    const url = URL.createObjectURL(blob)
    window.open(url, '_blank')
  } catch(err) {
    console.error(err)
  }
}

async function reporteIndividual() {
  try {
    const reporte = await fetch(server.HOST + '/api/v1/reportes/individual/' + ID.value, {
      method: 'GET'
    })
    if( reporte.ok ){
      const blob = await reporte.blob()
      const url = URL.createObjectURL(blob)
      window.open(url, '_blank')
    } else {
      toast.add({severity:'error', detail: 'Error al generar reporte', life: 3000 })
    }
    pedidoID.value = false
  } catch(err) {
    console.error(err)
  }
  ID.value = null
}
</script>