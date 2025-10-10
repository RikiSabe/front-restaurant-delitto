<template>
  <Toast />
  <div class="flex items-center justify-center h-screen">
    <Card style="width: 23rem; overflow: hidden;">
      <template #title>
        <p class="text-center">Inicio de Sesión</p>
      </template>
      <template #content>
        <div class="flex flex-col gap-1">
          <label for="usuario" class="text-sm text-slate-500"> Usuario </label>
          <InputText 
            id="usuario" name="usuario" placeholder="Ingrese su nombre de usuario"
            v-model="initialValues.usuario"
            size="small" fluid />
        </div>
        <div class="flex flex-col gap-1 mt-4">
          <label for="contra" class="text-sm text-slate-500"> Contraseña </label>
          <Password
            placeholder="Ingrese su contraseña" :feedback="false" 
            v-model="initialValues.contra"
            size="small" fluid />
        </div>
      </template>
      <template #footer>
        <div class="flex flex-col gap-2">
          <Button label="Iniciar Sesion" @click="iniciarSesion" fluid />
          <Button 
            label="ir a panel de cliente" fluid variant="link"
            @click="router.push('/cliente/pedido')"/>
        </div>
      </template>
    </Card>
  </div>
</template>

<script setup lang="ts">
import { server } from '~/server/server'

const initialValues = reactive({ usuario: '', contra: '' })
const router = useRouter()
const toast = useToast()

async function iniciarSesion() {
  if( !initialValues.usuario || !initialValues.contra ){
    toast.add({ severity: 'warn', summary: 'Datos incompletos', life: 3000 })
    return
  }
  try {
    const response:any = await $fetch(server.HOST + '/api/v1/login', {
      method: 'POST',
      body: JSON.stringify(initialValues)
    })
    if(response.rol === 'admin'){
      router.push('/admin/usuarios')
    } else 
    if(response.rol === 'cajero'){
      router.push('/cajero/pedido')
    }
  } catch (err) {
    toast.add({ severity: 'error', summary: 'Credenciales Incorrectas', life: 3000 })
    console.error(err)
  }
}
</script>