<template>
  <Dialog v-model:visible="visible" modal header="Agregar Nuevo Insumo" :style="{ width: '25rem' }">
    <Form 
      v-slot="$form" :resolver="resolver" 
      :initialValues="initialValues" @submit="onFormSubmit" 
      class="flex flex-col gap-2 w-full">
      
      <div class="flex flex-col gap-1">
        <label for="nombre"> Nombre </label>
        <InputText 
          id="nombre" name="nombre" 
          v-model="initialValues.nombre"
          placeholder="Ingrese un nombre" 
          fluid size="small" />
        <Message v-if="$form.nombre?.invalid" severity="error" size="small" variant="simple">
          {{ $form.nombre.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="stock_inicial"> Stock Inicial </label>
        <InputNumber 
          id="stock_inicial" name="stock_inicial"
          v-model="initialValues.stock_inicial"
          placeholder="Ingrese el stock inicial" 
          fluid size="small" />
        <Message v-if="$form.stock_inicial?.invalid" severity="error" size="small" variant="simple">
          {{ $form.stock_inicial.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="stock_minimo"> Stock Inicial </label>
        <InputNumber 
          id="stock_minimo" name="stock_minimo"
          v-model="initialValues.stock_minimo"
          placeholder="Ingrese el stock minimo" 
          fluid size="small" />
        <Message v-if="$form.stock_minimo?.invalid" severity="error" size="small" variant="simple">
          {{ $form.stock_minimo.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="proveedor"> Seleccione un proveedor </label>
        <Select 
          name="proveedor" :options="Proveedores" 
          v-model="initialValues.id_proveedor" 
          option-label="nombre" option-value="id"
          placeholder="Seleccione un proveedor"
          fluid size="small" />
        <Message v-if="$form.proveedor?.invalid" severity="error" size="small" variant="simple">
          {{ $form.proveedor.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="unidad_medida"> Unidad de Medida </label>
        <Select 
          id="unidad_medida" name="unidad_medida" 
          :options="UnidadesMedida" 
          placeholder="Seleccione una unidad de medida"
          v-model="initialValues.unidad_medida" 
          fluid size="small" />
        <Message v-if="$form.unidad_medida?.invalid" severity="error" size="small" variant="simple">
          {{ $form.unidad_medida.error?.message }}
        </Message>
      </div>

      <Button type="submit" label="Agregar" />
    </Form>
  </Dialog>
</template>

<script setup lang="ts">
import { zodResolver } from '@primevue/forms/resolvers/zod';
import { z } from 'zod';
import { server } from '~/server/server';

interface Props { open : boolean }
const props = defineProps<Props>()
const emit = defineEmits(['close', 'success', 'update', 'error'])
const visible = ref(props.open)

const UnidadesMedida = ref(['Litros', 'Kilos'])
const initialValues = reactive({ 
  nombre: '', 
  stock_inicial: 0,
  stock_minimo: 0, 
  id_proveedor: 0,
  unidad_medida: ''
})
const Proveedores = ref<any[]>([])

onMounted( async () => {
  const res:any[] = await $fetch(server.HOST + '/api/v1/proveedores', {
    method: 'GET'
  })
  Proveedores.value = res
})

watch(visible, (newValue) => { 
  if (!newValue) { emit('close') } 
})

const resolver = ref(zodResolver(
  z.object({
      nombre: z.string().min(1, { message: 'Nombre requerido.' }),
      precio: z.number().min(1, { message: 'Cantidad requerida.' }),
      categoria: z.union([
          z.object({
            name: z.string().min(1, 'Proveedor requerido.')
          }),
          z.any().refine((val) => true, { message: 'Proveedor requerido.' })
      ])
  })
))

async function onFormSubmit({ valid } : any ) {
  if( valid ){
    try{
      await $fetch(server.HOST + '/api/v1/insumos', {
        method: 'POST',
        headers: {
          "Content-Type" : "application/json"
        },
        body: initialValues
      })
      emit('update')
      emit('success')
      visible.value = false
    } catch(err) {
      console.error(err)
      emit('error')
    }
  }
}

</script>