<template>
  <Dialog v-model:visible="visible" modal header="Modificar Producto" :style="{ width: '25rem' }">
    <Form 
      v-slot="$form" :resolver="resolver" 
      :initialValues="initialValues" @submit="onFormSubmit" 
      class="flex flex-col gap-4 w-full">
      <div class="flex flex-col gap-1">
        <label for="nombre"> Nombre </label>
        <InputText 
          name="nombre" type="text" 
          v-model="initialValues.nombre"
          placeholder="nombre" fluid />
        <Message 
          v-if="$form.nombre?.invalid" 
          severity="error" size="small" variant="simple">
          {{ $form.nombre.error?.message }}
        </Message>
      </div>
      <div class="flex flex-col gap-1">
        <label for="precio"> Precio </label>
        <InputNumber 
          name="precio" type="text" 
          v-model="initialValues.precio"
          placeholder="precio" fluid />
        <Message 
          v-if="$form.precio?.invalid" 
          severity="error" size="small" variant="simple">
          {{ $form.precio.error?.message }}
        </Message>
      </div>
      <div class="flex flex-col gap-1">
        <label for="categoria"> Categoria </label>
        <Select 
          name="categoria" :options="Categorias" 
          v-model="initialValues.id_categoria" 
          option-label="nombre" option-value="id" />
        <Message 
          v-if="$form.categoria?.invalid" 
          severity="error" size="small" variant="simple">
          {{ $form.categoria.error?.message }}
        </Message>
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

const initialValues = reactive({ nombre: '', precio: 0, id_categoria: 0 })
const Categorias = ref<any[]>([])

onMounted( async () => {
  try {
    const resValue:any = await $fetch(server.HOST + '/api/v1/productos/' + props.id, {
      method: 'GET'
    })
    initialValues.nombre = resValue.nombre
    initialValues.precio = resValue.precio
    const categoria = resValue.categoria

    const res:any[] = await $fetch(server.HOST + '/api/v1/categorias', {
      method: 'GET'
    })
    Categorias.value = res

    const match = Categorias.value.find(cat => cat.nombre === categoria)
    initialValues.id_categoria = match ? match.id : null
  } catch(err) {
    console.error(err)
  }
})

watch(visible, (newValue) => { 
  if (!newValue) { emit('close') } 
})

const resolver = ref(zodResolver(
  z.object({
      nombre: z.string().min(1, { message: 'Nombre requerido.' }),
      precio: z.number().min(1, { message: 'Precio requerido.' }),
      categoria: z.union([
          z.object({
            name: z.string().min(1, 'Categoria requerida.')
          }),
          z.any().refine((val) => true, { message: 'Categoria requerida.' })
      ])
  })
))

async function onFormSubmit({ valid } : any ) {
  if( valid ){
    try{
      await $fetch(server.HOST + '/api/v1/productos/' + props.id, {
        method: 'PUT',
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