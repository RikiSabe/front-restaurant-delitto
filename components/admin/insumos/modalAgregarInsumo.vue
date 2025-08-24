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
          v-model="initialValues.stock_actual"
          placeholder="Ingrese el stock inicial" 
          fluid size="small" />
        <Message v-if="$form.stock_inicial?.invalid" severity="error" size="small" variant="simple">
          {{ $form.stock_inicial.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="stock_minimo"> Stock Minimo </label>
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

      <div class="flex flex-col gap-1">
        <label for="categoria"> Categoria </label>
        <Select 
          id="categoria" name="categoria"
          :options="Categorias"
          placeholder="Seleccione una categoria"
          option-label="nombre" option-value="id"
          v-model="initialValues.id_categoria"
          fluid size="small" />
        <Message v-if="$form.categoria?.invalid" severity="error" size="small" variant="simple">
          {{ $form.categoria.error?.message }}
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

const UnidadesMedida = ref(['litro', 'kg'])

const initialValues = reactive({ 
  nombre: '', 
  stock_actual: 0,
  stock_minimo: 0, 
  id_proveedor: 0,
  unidad_medida: '',
  id_categoria: 0
})

const Proveedores = ref<any[]>([])
const Categorias = ref<any[]>([])

onMounted( async () => {
  await ObtenerProveedores()
  await ObtenerCategorias()
})

const ObtenerProveedores = async () => {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/proveedores', {
      method: 'GET'
    })
    Proveedores.value = res
  } catch (err) {
    console.error(err)
  }
}

const ObtenerCategorias = async () => {
  try {
    const res:any[] = await $fetch(server.HOST + '/api/v1/categorias-insumos', {
      method: 'GET'
    })
    Categorias.value = res.map ( item => ({ nombre: item.nombre, id: item.id }))
  } catch (err) {
    console.error(err)
  }
}

watch(visible, (newValue) => { 
  if (!newValue) { emit('close') } 
})

const schema = z.object({
  nombre: z.string()
    .trim()
    .min(1, { message: 'Nombre requerido.' })
    .max(80, { message: 'Máximo 80 caracteres.' }),

  stock_inicial: z.coerce.number({ invalid_type_error: 'Stock inicial inválido.' })
    .int({ message: 'Debe ser un entero.' })
    .min(0, { message: 'Mínimo 0.' }),

  stock_minimo: z.coerce.number({ invalid_type_error: 'Stock mínimo inválido.' })
    .int({ message: 'Debe ser un entero.' })
    .min(0, { message: 'Mínimo 0.' }),

  proveedor: z.coerce.number({ invalid_type_error: 'Proveedor requerido.' })
    .int({ message: 'Proveedor requerido.' })
    .positive({ message: 'Proveedor requerido.' }),

  unidad_medida: z.string()
    .refine(v => UnidadesMedida.value.includes(v), {
      message: 'Unidad de medida requerida.',
    })
})

const resolver = ref(zodResolver(schema));

async function onFormSubmit({ valid } : any ) {
  if( valid ){
    console.log(initialValues)
    try{
      await $fetch(server.HOST + '/api/v1/insumos', {
        method: 'POST',
        headers: {
          "Content-Type" : "application/json"
        },
        body: initialValues
      })
      emit('update'), emit('success')
      visible.value = false
    } catch(err) {
      console.error(err)
      emit('error')
    }
  }
}

</script>