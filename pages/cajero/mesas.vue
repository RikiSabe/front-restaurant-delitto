<template>
  <Toast />
  <div class="p-2 rounded-lg mb-2">
    <p class="text-2xl" style="font-weight: 600;">Estado de las Mesas</p>
  </div>

  <div class="grid grid-cols-1 sm:grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4">
    <Card v-for="mesa in Mesas" :key="mesa.id" class="text-center" :class="{ 'bg-green-50': mesa.estado === 'Disponible', 'bg-red-50': mesa.estado === 'Ocupado' }">
      <template #content>
        <div class="flex flex-col items-center gap-3">
          <i class="pi pi-table text-6xl text-gray-400"></i>
          <p class="text-xl font-semibold">{{ mesa.nombre }}</p>
          <p class="text-sm text-gray-500">Capacidad: {{ mesa.capacidad }}</p>
          <span 
            class="status-badge"
            :class="mesa.estado === 'Disponible' ? 'status-disponible' : 'status-ocupado'"
          >
            {{ mesa.estado }}
          </span>
        </div>
      </template>
      <template #footer>
        <Button 
          v-if="mesa.estado == 'Ocupado'" 
          label="Liberar Mesa"
          @click="LiberarMesa(mesa.id)" 
          fluid size="small" 
          severity="secondary"
        />
      </template>
    </Card>
  </div>
</template>

<script setup lang="ts">
import { server } from '~/server/server'

definePageMeta({ layout: 'menu-cajero' })

const Mesas = ref<any[]>([])
const toast = useToast()

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

const LiberarMesa = async (id: any) => {
  try {
    await $fetch(server.HOST + '/api/v1/mesas/liberar/' + id, {
      method: 'PUT'
    })
    toast.add({severity: 'success', summary: 'Mesa liberada Exitosamente', life: 3000 })
    ObtenerMesas()

  } catch(err) { 
    console.error(err)
  }
}
</script>