<template>
  <Toast />
  <div class="p-2 rounded-lg mb-2">
    <p class="text-xl font-bold">Mesas</p>
  </div>

  <div class="grid grid-cols-4 gap-2">
    <Card v-for="mesa in Mesas" :key="mesa.id">
      <template #header>
        <p class="text-center p-4 text-xl"> {{ mesa.nombre }} </p>
      </template>
      <template #content>
        <p class="text-center p-4"> Capacidad de {{ mesa.capacidad }}</p>
        <p class="text-center"> {{ mesa.estado }} </p>
      </template>
      <template #footer>
        <Button 
          v-if="mesa.estado == 'Ocupado'" 
          label="Liberar"
          @click="LiberarMesa(mesa.id)" 
          fluid size="small" />
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