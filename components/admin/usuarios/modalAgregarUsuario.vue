<template>
  <Dialog v-model:visible="visible" modal header="Agregar Nuevo Usuario" :style="{ width: '25rem' }">
    <Form 
      v-slot="$form" :resolver="resolver" 
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
          <label for="celular"> Ingrese número de celular </label>
          <InputText 
            id="celular" name="celular" type="text"
            v-model="initialValues.celular"
            placeholder="Número de celular"
            maxlength="8"
            fluid size="small"/>
          <Message
            v-if="$form.celular?.invalid" 
            severity="error" size="small" variant="simple">
            {{ $form.celular.error?.message }}
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
          <InputGroup>
            <InputText 
              id="contra" name="contra" 
              :type="showPassword ? 'text' : 'password'"
              v-model="initialValues.contra"
              placeholder="contraseña" 
              fluid size="small"/>
            <InputGroupAddon class="cursor-pointer" @click="showPassword = !showPassword">
              <i class="pi" :class="showPassword ? 'pi-eye-slash' : 'pi-eye'"></i>
            </InputGroupAddon>
          </InputGroup>
          <Message
            v-if="$form.contra?.invalid" 
            severity="error" size="small" variant="simple">
            {{ $form.contra.error?.message }}
          </Message>
        </div>

        <div class="flex flex-col gap-1">
          <label for="confirmarContra"> Confirmar contraseña </label>
          <InputGroup>
            <InputText 
              id="confirmarContra" name="confirmarContra" 
              :type="showConfirmPassword ? 'text' : 'password'"
              v-model="initialValues.confirmarContra"
              placeholder="Confirmar contraseña" 
              fluid size="small"/>
            <InputGroupAddon class="cursor-pointer" @click="showConfirmPassword = !showConfirmPassword">
              <i class="pi" :class="showConfirmPassword ? 'pi-eye-slash' : 'pi-eye'"></i>
            </InputGroupAddon>
          </InputGroup>
          <Message
            v-if="$form.confirmarContra?.invalid" 
            severity="error" size="small" variant="simple">
            {{ $form.confirmarContra.error?.message }}
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
        
        <Button type="submit" label="Agregar" />
      </div>
    </Form>
  </Dialog>
</template>

<script setup lang="ts">
import { zodResolver } from '@primevue/forms/resolvers/zod';
import { z } from 'zod';
import { server } from '~/server/server';
import InputGroup from 'primevue/inputgroup';
import InputGroupAddon from 'primevue/inputgroupaddon';

interface Props { open : boolean }
const props = defineProps<Props>()
const emit = defineEmits(['close', 'success', 'update', 'error'])
const visible = ref(props.open)

const ciExistente = ref(false)
const usuarioExistente = ref(false)
const usuarios = ref<any[]>([])
const showPassword = ref(false);
const showConfirmPassword = ref(false);

const initialValues = reactive({ 
  nombre: '', 
  apellido: '',
  ci: '',
  celular: '',
  usuario: '',
  contra: '',
  confirmarContra: '',
  rol: '',
  estado: 'Activo'
})
const Roles = ref(['admin', 'cajero'])

onMounted(async () => {
  const res:any[] = await $fetch(server.HOST + '/api/v1/usuarios', {
    method: 'GET'
  })
  usuarios.value = res
})

watch(() => props.open, (newValue) => {
  visible.value = newValue
})

watch(visible, (newValue) => { 
  if (!newValue) { emit('close') } 
})

const resolver = ref(zodResolver(
  z.object({
    nombre: z.string().min(1, { message: 'Nombre requerido.' }),
    apellido: z.string().min(1, {message: 'Apellido requerido.' }),
    ci: z.string()
      .min(7, {message: 'CI requerido entre 7 a 9 caracteres.'})
      .max(9, {message: 'CI requerido entre 7 a 9 caracteres.'}),
    celular: z.string().length(8, { message: 'El número de celular debe tener 8 dígitos.' }),
    usuario: z.string().min(1, {message: 'Usuario requerido.'}),
    contra: z.string()
      .min(8, { message: 'La contraseña debe tener al menos 8 caracteres.' })
      .regex(/^(?=.*[a-zA-Z])(?=.*[0-9])/, { message: 'La contraseña debe contener letras y números.' }),
    confirmarContra: z.string()
      .min(8, { message: 'La contraseña debe tener al menos 8 caracteres.' })
      .regex(/^(?=.*[a-zA-Z])(?=.*[0-9])/, { message: 'La contraseña debe contener letras y números.' }),
    rol: z.union([
      z.object({
        name: z.string().min(1, 'Rol requerido.')
      }),
      z.any().refine((val) => true, { message: 'Rol requerido.' })
    ])
  }).refine(data => data.contra === data.confirmarContra, {
    message: "Las contraseñas no coinciden",
    path: ["confirmarContra"],
  })
))

async function onFormSubmit({ valid } : any ) {
  if( valid ){
    const ci = initialValues.ci.trim().toLowerCase()
    const username = initialValues.usuario.trim().toLowerCase()

    const ciDuplicado = usuarios.value.find(u => u.ci.toLowerCase() === ci)
    const usuarioDuplicado = usuarios.value.find(u => u.usuario.toLowerCase() === username)

    ciExistente.value = !!ciDuplicado
    usuarioExistente.value = !!usuarioDuplicado

    if (ciDuplicado || usuarioDuplicado) {
      return
    }

    try{
      // Exclude confirmarContra from the body sent to the server
      const { confirmarContra, ...userData } = initialValues;

      await $fetch(server.HOST + '/api/v1/usuarios', {
        method: 'POST',
        body: userData
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
