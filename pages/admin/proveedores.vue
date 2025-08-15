<template>
  <Toast />
  <div class="p-2 rounded-lg mb-2">
    <p class="text-xl font-bold">Proveedores</p>
  </div>
  <div>
    <DataTable 
      :value="Proveedores" tableStyle="min-width: 50rem" 
      show-gridlines size="small"
      paginator :rows="5">
      <template #header>
        <div class="flex justify-end">
          <Button label="Agregar Proveedor" size="small" @click="AgregarProveedor = true"/>
        </div>
      </template>
      <template #empty>
        <p class="text-center"> No hay Proveedores </p>
      </template>
      <template #paginatorcontainer="{ first, last, prevPageCallback, nextPageCallback, totalRecords }">
        <div class="flex items-center gap-2 bg-transparent w-full py-1 px-2 justify-between">
          <Button icon="pi pi-chevron-left" rounded text @click="prevPageCallback" />
          <div class="text-color font-medium">
            <span>Viendo {{ first }} a {{ last }} de {{ totalRecords }} proveedores </span>
          </div>
          <Button icon="pi pi-chevron-right" rounded text @click="nextPageCallback" />
        </div>
      </template>
      <Column field="id" header="ID" />
      <Column field="nombre" header="Nombre" />
      <Column field="telefono" header="Telefono" />
      <Column field="correo" header="Correo" />
      <Column field="direccion" header="Dirección" /> 
      <Column field="estado" header="Estado" /> 
      <Column header="Acciones">
        <template #body="slotProps">
          <div class="flex items-center justify-center">
            <Button 
              label="Editar" variant="text" 
              @click="idProveedor = slotProps.data.id, ModificarProveedor = true"/>
          </div>
        </template>
      </Column>
    </DataTable>
  </div>

  <modalAgregarProveedor 
    :open="AgregarProveedor"
    v-if="AgregarProveedor"
    @close="AgregarProveedor = false"
    @update="obtenerProveedores"
    @success="toast.add({ severity: 'success', summary: 'Agregado Exitoso', life: 3000 })"
    @error="toast.add({ severity: 'error', summary: 'Error al Agregar', life: 3000 })" />

  <modalModificarProveedor 
    :open="ModificarProveedor"
    :id="idProveedor"
    v-if="ModificarProveedor"
    @close="ModificarProveedor = false"
    @update="obtenerProveedores"
    @success="toast.add({ severity: 'success', summary: 'Modificado Exitoso', life: 3000 })"
    @error="toast.add({ severity: 'error', summary: 'Error al Modificar', life: 3000 })" />
</template>

<script setup lang="ts">
import modalAgregarProveedor from '~/components/admin/proveedores/modalAgregarProveedor.vue'
import modalModificarProveedor from '~/components/admin/proveedores/modalModificarProveedor.vue'
import { server } from '~/server/server'

definePageMeta({ layout : 'menu-admin' })

const toast = useToast()
const Proveedores = ref<any[]>([])
const AgregarProveedor = ref(false)
const ModificarProveedor = ref(false)
const idProveedor = ref(0)

onMounted( async () => {
  await obtenerProveedores()
})

async function obtenerProveedores() {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/proveedores', {
      method: 'GET'
    })
    Proveedores.value = res
  } catch (err) {
    console.error(err)
  }
}
</script>