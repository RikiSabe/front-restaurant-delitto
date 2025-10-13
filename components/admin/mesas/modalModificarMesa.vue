<template>
<Dialog v-model:visible="visible" modal header="Modificar Mesa" :style="{ width: '25rem' }">
    <Form 
      v-slot="$form" :resolver="resolver" ref="validations"
      :initialValues="initialValues" @submit="onFormSubmit" 
      class="flex flex-col gap-4 w-full">

      <div class="grid grid-cols-1 gap-2">
         <!-- Nombre Mesa -->
        <div class="flex flex-col gap-1">
          <label for="nombre"> Nombre </label>
          <InputText 
            id="nombre" name="nombre"
            v-model="initialValues.nombre"
            placeholder="Ingrese nombre" 
            fluid size="small"
            :class="{ 'p-invalid': nombreExistente }"
          />
          <Message
            v-if="$form.nombre?.invalid"
            severity="error" size="small" variant="simple">
            {{ $form.nombre.error?.message }}
          </Message>
          <Message v-if="nombreExistente" severity="error" size="small" variant="simple">
            Este nombre ya existe
          </Message>
        </div>

        <!-- Capacidad Mesa -->
        <div class="flex flex-col gap-1">
          <label for="capacidad"> Ingrese la cantidad </label>
          <InputNumber 
            id="capacidad" name="capacidad" 
            :min="0" :max="24" :step="1"
            v-model="initialValues.capacidad"
            placeholder="capacidad" 
            fluid size="small"/>
          <Message
            v-if="$form.capacidad?.invalid"
            severity="error" size="small" variant="simple">
            {{ $form.capacidad.error?.message }}
          </Message>
        </div>

        <div class="flex flex-col gap-1">
          <label for="estado"> Estado </label>
          <Select 
            id="estado" name="estado"
            placeholder="Seleccione su estado"
            :options="Estados" v-model="initialValues.estado" 
            fluid size="small"/>
          <Message
            v-if="$form.estado?.invalid"
            severity="error" size="small" variant="simple">
            {{ $form.estado.error?.message }}
          </Message>
        </div>

        <Button type="submit" label="Modificar" />
      </div>
    </Form>
  </Dialog>  
</template>

<script setup lang="ts">
import { zodResolver } from '@primevue/forms/resolvers/zod';
import { z } from 'zod';
import { server } from '~/server/server';

interface Props { open: boolean, id: number}
const props = defineProps<Props>()
const emit = defineEmits(['close', 'update', 'success', 'fail'])
const visible = ref(props.open)
const validations = ref()

const nombreExistente = ref(false)
const mesas = ref<any[]>([])

const initialValues = reactive({
  nombre: '',
  estado: '',
  capacidad: 0
})

const Estados = ref(['Disponible', 'Ocupado'])

watch(() => props.open, (newValue) => {
  visible.value = newValue
})

watch((visible), newValue => {
  if( !newValue ) {
    emit('close')
  }
})

onMounted( async () => {
  await ObtenerMesa()
  const res:any[] = await $fetch(server.HOST + '/api/v1/mesas', {
    method: 'GET'
  })
  mesas.value = res
})

const ObtenerMesa = async () => {
  const res:any = await $fetch(server.HOST + '/api/v1/mesas/' + props.id, {
    method: 'GET'
  })
  Object.assign(initialValues, res)
  validations.value?.reset()
}

const resolver = ref(zodResolver(
  z.object({
    nombre: z.string()
    .trim()
    .min(1, { message: 'Nombre requerido.' })
    .max(60, { message: 'Máximo 60 caracteres.' }),

  capacidad: z.coerce.number({
      invalid_type_error: 'Capacidad inválida.',
    })
    .int({ message: 'Debe ser un entero.' })
    .min(1, { message: 'Mínimo 1.' })
    .max(24, { message: 'Máximo 24.' }),

  estado: z.string()
    .refine((v) => (Estados.value).includes(v), {
      message: 'Estado requerido.',
    }),
  })
))

async function onFormSubmit({ valid } : any) {
  if ( valid ) {
    const nombreMesa = initialValues.nombre.trim().toLowerCase()
    const mesaExistente = mesas.value.find(m => m.nombre.toLowerCase() === nombreMesa && m.id !== props.id)

    if (mesaExistente) {
      nombreExistente.value = true
      return
    }

    nombreExistente.value = false
    try {
      await $fetch(server.HOST + '/api/v1/mesas/' + props.id, {
        method: 'PUT',
        body: initialValues
      })
      emit('success'), emit('update')
      visible.value = false
    } catch (err) {
      console.error(err)
      emit('fail')
    }
  }
}
</script>