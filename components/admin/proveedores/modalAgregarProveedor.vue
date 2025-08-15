<template>
  <Dialog v-model:visible="visible" modal header="Agregar Nuevo Proveedor" :style="{ width: '25rem' }">
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
        <label for="telefono"> Telefono </label>
        <InputText 
          id="telefono" name="telefono"
          v-model="initialValues.telefono"
          placeholder="Ingrese un telefono" 
          fluid size="small" />
        <Message v-if="$form.telefono?.invalid" severity="error" size="small" variant="simple">
          {{ $form.telefono.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="correo"> Correo Electronico </label>
        <InputText 
          id="correo" name="correo" type="text" 
          v-model="initialValues.correo"
          placeholder="Ingrese un correo" 
          fluid size="small" />
        <Message v-if="$form.correo?.invalid" severity="error" size="small" variant="simple">
          {{ $form.correo.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="direccion"> Direcion </label>
        <InputText 
          id="direccion" name="direccion"
          v-model="initialValues.direccion"
          placeholder="Ingrese una direccion" 
          fluid size="small"/>
        <Message v-if="$form.direccion?.invalid" severity="error" size="small" variant="simple">
          {{ $form.direccion.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="estado"> Estado </label>
        <Select
          id="estado" name="estado"
          v-model="initialValues.estado"
          placeholder="Seleccione un estado" 
          :options="Estados"
          fluid size="small" />
        <Message v-if="$form.estado?.invalid" severity="error" size="small" variant="simple">
          {{ $form.estado.error?.message }}
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

const Estados = ref(['Activo', 'Inactivo'])

const initialValues = reactive({ 
  nombre: '',
  telefono: '',
  correo: '',
  direccion: '',
  estado: '',
})

watch(visible, (newValue) => { 
  if (!newValue) { emit('close') } 
})

const resolver = ref(zodResolver(
  z.object({
    nombre: z
      .string()
      .min(1, { message: 'Nombre requerido.' })
      .max(100, { message: 'El nombre no debe superar los 100 caracteres.' }),

    telefono: z
      .string()
      .min(1, { message: 'Teléfono requerido.' })
      .regex(/^\d+$/, { message: 'El teléfono solo debe contener números.' })
      .min(8, { message: 'El teléfono debe tener al menos 8 dígitos.' })
      .max(8, { message: 'El teléfono no puede tener más de 8 dígitos.' }),

    correo: z
      .string()
      .min(1, { message: 'Correo requerido.' })
      .email({ message: 'Debe ingresar un correo válido.' }),

    direccion: z
      .string()
      .min(1, { message: 'Dirección requerida.' })
      .max(100, { message: 'La dirección no debe superar los 100 caracteres.' }),

    estado: z
      .enum(['Activo', 'Inactivo'], { message: 'Debe seleccionar un estado válido.' })
  })
))


async function onFormSubmit({ valid } : any ) {
  if( valid ){
    try{
      await $fetch(server.HOST + '/api/v1/proveedores', {
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