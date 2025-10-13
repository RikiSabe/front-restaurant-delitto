<template>
  <Dialog v-model:visible="visible" modal header="Modificar Usuario" :style="{ width: '25rem' }">
    <Form 
      v-slot="$form" :resolver="resolver" ref="validations" 
      :initialValues="initialValues" @submit="onFormSubmit" 
      class="flex flex-col gap-4 w-full">

      <div class="grid grid-cols-1 gap-2">
        <div class="flex flex-col gap-1">
          <label for="nombre"> Ingrese un nombre </label>
          <InputText 
            id="nombre" name="nombre" type="text"
            v-model="initialValues.nombre"
            placeholder="nombre" 
            fluid size="small"/>
          <Message
            v-if="$form.nombre?.invalid"
            severity="error" size="small" variant="simple">
            {{ $form.nombre.error?.message }}
          </Message>
        </div>

        <div class="flex flex-col gap-1">
          <label for="apellido"> Ingrese un apellido </label>
          <InputText 
            id="apellido" name="apellido" type="text"
            v-model="initialValues.apellido"
            placeholder="apellido" 
            fluid size="small"/>
          <Message
            v-if="$form.apellido?.invalid" 
            severity="error" size="small" variant="simple">
            {{ $form.apellido.error?.message }}
          </Message>
        </div>

        <div class="flex flex-col gap-1">
          <label for="ci"> Ingrese un ci </label>
          <InputText 
            id="ci" name="ci" type="text"
            v-model="initialValues.ci"
            placeholder="ci" 
            fluid size="small"
            :class="{ 'p-invalid': ciExistente }"
          />
          <Message
            v-if="$form.ci?.invalid" 
            severity="error" size="small" variant="simple">
            {{ $form.ci.error?.message }}
          </Message>
          <Message v-if="ciExistente" severity="error" size="small" variant="simple">
            Este CI ya existe
          </Message>
        </div>

        <div class="flex flex-col gap-1">
          <label for="usuario"> Ingrese un usuario </label>
          <InputText 
            id="usuario" name="usuario" type="text"
            v-model="initialValues.usuario"
            placeholder="usuario" 
            fluid size="small"
            :class="{ 'p-invalid': usuarioExistente }"
          />
          <Message
            v-if="$form.usuario?.invalid" 
            severity="error" size="small" variant="simple">
            {{ $form.usuario.error?.message }}
          </Message>
          <Message v-if="usuarioExistente" severity="error" size="small" variant="simple">
            Este usuario ya existe
          </Message>
        </div>

        <div class="flex flex-col gap-1">
          <label for="contra"> Ingrese una contraseña </label>
          <InputText 
            id="contra" name="contra" type="text"
            v-model="initialValues.contra"
            placeholder="contraseña" 
            fluid size="small"/>
          <Message
            v-if="$form.contra?.invalid" 
            severity="error" size="small" variant="simple">
            {{ $form.contra.error?.message }}
          </Message>
        </div>

        <div class="flex flex-col gap-1">
          <label for="rol"> Seleccione un rol </label>
          <Select 
            name="rol" :options="Roles" 
            v-model="initialValues.rol" 
            placeholder="Seleccione un rol"
            fluid size="small" />
          <Message 
            v-if="$form.rol?.invalid" 
            severity="error" size="small" variant="simple">
            {{ $form.rol.error?.message }}
          </Message>
        </div>

        <div class="flex flex-col gap-1">
          <label for="estado"> Seleccione un estado </label>
          <Select
            name="estado" :options="Estados"
            v-model="initialValues.estado"
            placeholder="Seleccion un estado"
            fluid size="small"/>
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

interface Props { open : boolean, id: number }
const props = defineProps<Props>()
const emit = defineEmits(['close', 'success', 'update', 'error'])
const visible = ref(props.open)
const validations = ref()

const ciExistente = ref(false)
const usuarioExistente = ref(false)
const usuarios = ref<any[]>([])

const initialValues = reactive({ 
  nombre: '', 
  apellido: '',
  ci: '',
  usuario: '',
  contra: '',
  rol: '',
  estado: ''
})

const Roles = ref(['admin', 'cajero'])
const Estados = ref(['Activo', 'Inactivo'])

onMounted( async () => {
  try {
    const resValue:any = await $fetch(server.HOST + '/api/v1/usuarios/' + props.id, {
      method: 'GET'
    })
    Object.assign(initialValues, resValue)
    validations.value?.reset()

    const res:any[] = await $fetch(server.HOST + '/api/v1/usuarios', {
      method: 'GET'
    })
    usuarios.value = res
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

const getSchema = () => z.object({
  nombre: z.string().min(1, { message: 'Nombre requerido.' }),
  apellido: z.string().min(1, {message: 'Apellido requerido.' }),
  ci: z.string()
    .min(7, {message: 'CI requerido entre 7 a 9 caracteres.'})
    .max(9, {message: 'CI requerido entre 7 a 9 caracteres.'}),
  usuario: z.string().min(1, {message: 'Usuario requerido.'}),
  contra: z.string().min(5, {message: 'Contraseña con 6 caracteres minimos.'}),
  rol: z.union([
    z.object({
      name: z.string().min(1, 'Rol requerido.')
    }),
    z.any().refine((val) => true, { message: 'Rol requerido.' })
  ])
})

const resolver = computed( () => zodResolver(getSchema()))

async function onFormSubmit({ valid } : any ) {
  if( valid ){
    const ci = initialValues.ci.trim().toLowerCase()
    const username = initialValues.usuario.trim().toLowerCase()

    const ciDuplicado = usuarios.value.find(u => u.ci.toLowerCase() === ci && u.id !== props.id)
    const usuarioDuplicado = usuarios.value.find(u => u.usuario.toLowerCase() === username && u.id !== props.id)

    ciExistente.value = !!ciDuplicado
    usuarioExistente.value = !!usuarioDuplicado

    if (ciDuplicado || usuarioDuplicado) {
      return
    }

    try{
      await $fetch(server.HOST + '/api/v1/usuarios/' + props.id, {
        method: 'PUT',
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