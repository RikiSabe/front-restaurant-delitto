<template>
  <Dialog v-model:visible="visible" modal header="Modificar Usuario" :style="{ width: '25rem' }">
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
        <label for="rol"> Roles </label>
        <Select 
          name="rol" :options="Roles" 
          v-model="initialValues.rol" />
        <Message 
          v-if="$form.rol?.invalid" 
          severity="error" size="small" variant="simple">
          {{ $form.rol.error?.message }}
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

const initialValues = reactive({ nombre: '', rol: '' })
const Roles = ref(['cliente'])

onMounted( async () => {
  try {
    const resValue:any = await $fetch(server.HOST + '/api/v1/usuarios/' + props.id, {
      method: 'GET'
    })
    initialValues.nombre = resValue.nombre
    initialValues.rol = resValue.rol
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
      categoria: z.union([
          z.object({
            name: z.string().min(1, 'Rol requerido.')
          }),
          z.any().refine((val) => true, { message: 'Rol requerido.' })
      ])
  })
))

async function onFormSubmit({ valid } : any ) {
  if( valid ){
    try{
      await $fetch(server.HOST + '/api/v1/usuarios/' + props.id, {
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