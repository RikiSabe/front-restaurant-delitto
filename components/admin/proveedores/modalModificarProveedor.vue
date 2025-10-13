<template>
  <Dialog v-model:visible="visible" modal header="Modificar Proveedor" :style="{ width: '25rem' }">
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
          fluid size="small" 
          :class="{ 'p-invalid': nombreExistente }"
        />
        <Message v-if="$form.nombre?.invalid" severity="error" size="small" variant="simple">
          {{ $form.nombre.error?.message }}
        </Message>
        <Message v-if="nombreExistente" severity="error" size="small" variant="simple">
          Este nombre ya existe
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

const Estados = ref(['Activo', 'Inactivo'])
const nombreExistente = ref(false)
const proveedores = ref<any[]>([])

const initialValues = reactive({ 
  nombre: '',
  telefono: '',
  correo: '',
  direccion: '',
  estado: '', 
})

onMounted( async () => {
  try {
    const resValue:any = await $fetch(server.HOST + '/api/v1/proveedores/' + props.id, {
      method: 'GET'
    })
    initialValues.nombre = resValue.nombre
    initialValues.telefono = resValue.telefono
    initialValues.correo = resValue.correo
    initialValues.direccion = resValue.direccion
    initialValues.estado = resValue.estado

    const res:any[] = await $fetch(server.HOST + '/api/v1/proveedores', {
      method: 'GET'
    })
    proveedores.value = res
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

const resolver = ref(zodResolver(
  z.object({
    nombre: z.string(),
    telefono: z.string().regex(/^$|^\d{8}$/, { message: 'El teléfono debe tener 8 dígitos.' }),
    correo: z.string().email({ message: 'Correo inválido' }).or(z.literal('')),
    direccion: z.string(),
    estado: z.enum(['Activo', 'Inactivo'])
  })
))

async function onFormSubmit({ valid } : any ) {
  if( valid ){
    const nombreProveedor = initialValues.nombre.trim().toLowerCase()
    const proveedorExistente = proveedores.value.find(p => p.nombre.toLowerCase() === nombreProveedor && p.id !== props.id)

    if (proveedorExistente) {
      nombreExistente.value = true
      return
    }

    nombreExistente.value = false
    try{
      await $fetch(server.HOST + '/api/v1/proveedores/' + props.id, {
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