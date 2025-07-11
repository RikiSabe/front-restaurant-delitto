<template>
  <Toast />
  <div class="p-2 rounded-lg mb-2">
    <p class="text-xl font-bold">Usuarios</p>
  </div>
  <div>
    <DataTable 
      :value="Usuarios" tableStyle="min-width: 70rem" 
      show-gridlines size="small"
      paginator :rows="5">
      <template #header>
        <div class="flex justify-end">
          <Button label="Agregar Usuario" size="small" @click="AgregarUsuario = true"/>
        </div>
      </template>
      <template #empty>
        <p class="text-center"> No hay Usuarios </p>
      </template>
      <template #paginatorcontainer="{ first, last, prevPageCallback, nextPageCallback, totalRecords }">
        <div class="flex items-center gap-2 bg-transparent w-full py-1 px-2 justify-between">
          <Button icon="pi pi-chevron-left" rounded text @click="prevPageCallback" />
          <div class="text-color font-medium">
            <span>Viendo {{ first }} a {{ last }} de {{ totalRecords }} usuarios </span>
          </div>
          <Button icon="pi pi-chevron-right" rounded text @click="nextPageCallback" />
        </div>
      </template>
      <Column field="id" header="ID" />
      <Column field="nombre" header="Nombre" />
      <Column field="rol" header="Rol" />
      <Column header="Acciones">
        <template #body="slotProps">
          <div class="flex items-center justify-center">
            <Button label="Editar" variant="text" @click="idUsuario = slotProps.data.id, ModificarUsuario = true"/>
          </div>
        </template>
      </Column>
    </DataTable>
  </div>

  <modalAgregarUsuario 
    :open="AgregarUsuario"
    v-if="AgregarUsuario"
    @close="AgregarUsuario = false"
    @update="obtenerUsuarios"
    @success="toast.add({ severity: 'success', summary: 'Agregado Exitoso', life: 3000 })"
    @error="toast.add({ severity: 'error', summary: 'Error al Agregar', life: 3000 })" />

  <modalModificarUsuario 
    :open="ModificarUsuario"
    :id="idUsuario"
    v-if="ModificarUsuario"
    @close="ModificarUsuario = false"
    @update="obtenerUsuarios"
    @success="toast.add({ severity: 'success', summary: 'Modificado Exitoso', life: 3000 })"
    @error="toast.add({ severity: 'error', summary: 'Error al Modificar', life: 3000 })" />
</template>

<script setup lang="ts">
import modalAgregarUsuario from '~/components/admin/usuarios/modalAgregarUsuario.vue'
import modalModificarUsuario from '~/components/admin/usuarios/modalModificarUsuario.vue'
import { server } from '~/server/server'

definePageMeta({ layout : 'menu-admin' })

const toast = useToast()
const Usuarios = ref<any[]>([])
const AgregarUsuario = ref(false), ModificarUsuario = ref(false)
const idUsuario = ref(0)

onMounted( async () => {
  await obtenerUsuarios()
})

async function obtenerUsuarios() {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/usuarios', {
      method: 'GET'
    })
    Usuarios.value = res
  } catch (err) {
    console.error(err)
  }
}
</script>