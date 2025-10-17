<template>
  <Dialog v-model:visible="visible" modal header="Modificar Proveedor" :style="{ width: '25rem' }">
    <Form 
      v-slot="$form" :resolver="resolver" 
      :initialValues="initialValues" @submit="onFormSubmit" 
      class="flex flex-col gap-2 w-full">
      
      <div class="flex flex-col gap-1">
        <label for="tipo">Tipo de Proveedor</label>
        <Select
          id="tipo" name="tipo"
          v-model="initialValues.tipo"
          placeholder="Seleccione un tipo" 
          :options="Tipos"
          fluid size="small" />
      </div>

      <div v-if="initialValues.tipo === 'Persona particular'">
        <div class="flex flex-col gap-1">
          <label for="ci">CI</label>
          <InputText 
            id="ci" name="ci"
            v-model="initialValues.ci"
            placeholder="Ingrese el CI" 
            fluid size="small"/>
        </div>
      </div>

      <div v-if="initialValues.tipo === 'Empresa'">
        <div class="flex flex-col gap-1">
          <label for="nit">NIT</label>
          <InputText 
            id="nit" name="nit"
            v-model="initialValues.nit"
            placeholder="Ingrese el NIT" 
            fluid size="small"/>
        </div>
        <div class="flex flex-col gap-1">
          <label for="nombre_encargado">Nombre del encargado</label>
          <InputText 
            id="nombre_encargado" name="nombre_encargado"
            v-model="initialValues.nombre_encargado"
            placeholder="Ingrese el nombre del encargado" 
            fluid size="small" />
        </div>
      </div>

      <Button type="submit" label="Modificar" />
    </Form>
  </Dialog>
</template>

<script setup lang="ts">
import { zodResolver } from '@primevue/forms/resolvers/zod';
import { z } from 'zod';
import { server } from '~/server/server';

interface Props { open : boolean, id: number }
const props = defineProps<Props>()
const emit = defineEmits(['close', 'success', 'update', 'error'])
const visible = ref(props.open)

const Tipos = ref(['Persona particular', 'Empresa'])

const initialValues = reactive({ 
  tipo: 'Persona particular',
  ci: '',
  nit: '',
  nombre_encargado: '',
})

onMounted( async () => {
  try {
    const resValue:any = await $fetch(server.HOST + '/api/v1/proveedores/' + props.id, {
      method: 'GET'
    })
    initialValues.tipo = resValue.tipo === 'persona' ? 'Persona particular' : 'Empresa'
    initialValues.ci = resValue.ci
    initialValues.nit = resValue.nit
    initialValues.nombre_encargado = resValue.nombre_encargado
  } catch(err) {
    console.error(err)
  }
})

watch(() => props.open, (newValue) => {
  visible.value = newValue
})

watch(visible, (newValue) => { 
  if (!newValue) { emit('close') } 
})

const resolver = computed(() => {
  if (initialValues.tipo === 'Persona particular') {
    return zodResolver(z.object({
      tipo: z.literal('Persona particular'),
      ci: z.string().min(1, { message: 'CI requerido.' }),
      nit: z.string().optional(),
      nombre_encargado: z.string().optional(),
    }));
  }

  return zodResolver(z.object({
    tipo: z.literal('Empresa'),
    nit: z.string().min(1, { message: 'NIT requerido.' }),
    nombre_encargado: z.string().min(1, { message: 'Nombre del encargado requerido.' }),
    ci: z.string().optional(),
  }));
});

async function onFormSubmit({ valid } : any ) {
  if( valid ){
    // ...
  }
}
</script>
