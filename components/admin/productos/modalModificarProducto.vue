<template>
  <Dialog v-model:visible="visible" modal header="Modificar Producto" :style="{ width: '25rem' }">
    <Form 
      v-slot="$form" :resolver="resolver" 
      :initialValues="initialValues" @submit="onFormSubmit" 
      class="flex flex-col gap-2 w-full">
      
      <div class="flex flex-col gap-1">
        <label for="nombre"> Nombre </label>
        <InputText 
          id="nombre" name="nombre" type="text" 
          v-model="initialValues.nombre"
          placeholder="Ingrese un nombre" 
          fluid size="small" 
          :class="{ 'p-invalid': nombreError }">
        </InputText>
        <Message v-if="nombreError" severity="error" size="small" variant="simple">
          {{ nombreError }}
        </Message>
        <Message v-if="$form.nombre?.invalid && !nombreError" severity="error" size="small" variant="simple">
          {{ $form.nombre.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="precio"> Precio </label>
        <InputNumber 
          id="precio" name="precio"
          v-model="initialValues.precio"
          placeholder="Ingrese un precio" 
          fluid size="small" />
        <Message v-if="$form.precio?.invalid" severity="error" size="small" variant="simple">
          {{ $form.precio.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="descripcion"> Descripcion </label>
        <InputText
          id="descripcion" name="descripcion"
          v-model="initialValues.descripcion"
          placeholder="Ingrese una descripcion" 
          fluid size="small" />
        <Message v-if="$form.descripcion?.invalid" severity="error" size="small" variant="simple">
          {{ $form.descripcion.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="categoria"> Categoria </label>
        <Select 
          name="categoria" :options="Categorias" 
          v-model="initialValues.id_categoria"   
          placeholder="Seleccione una categoria" 
          option-label="nombre" option-value="id" 
          fluid size="small" />
        <Message v-if="$form.categoria?.invalid" severity="error" size="small" variant="simple">
          {{ $form.categoria.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="estado"> Estado </label>
        <Select 
          id="estado" name="estado" :options="Estados" 
          v-model="initialValues.estado"   
          placeholder="Seleccione un estado"
          fluid size="small" />
        <Message v-if="$form.estado?.invalid" severity="error" size="small" variant="simple">
          {{ $form.estado.error?.message }}
        </Message>
      </div>

      <div class="flex flex-col gap-1">
        <label for="foto"> Foto </label>
        <div class="flex flex-col gap-1 items-center justify-center">
          <FileUpload 
            id="foto" name="foto"
            mode="basic" @select="onFileSelect" 
            severity="secondary" class="p-button-outlined" 
            choose-label="Seleccione una imagen" accept="image/*"
            custom-upload auto />
          <img 
            v-if="src" :src="src"
            alt="Image" class="shadow-md rounded-xl w-64 h-64"  />
          <Message v-if="$form.foto?.invalid" severity="error" size="small" variant="simple">
            {{ $form.foto.error?.message }}
          </Message>
        </div>
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
const src = ref<any>(null)

const initialValues = reactive({ 
  nombre: '', 
  precio: 0, 
  id_categoria: 0,
  descripcion: '',
  estado: '',
  foto: null as File | null
})
const Categorias = ref<any[]>([])

onMounted( async () => {
  try {
    const resValue:any = await $fetch(server.HOST + '/api/v1/productos/' + props.id, {
      method: 'GET'
    })
    initialValues.nombre = resValue.nombre
    initialValues.precio = resValue.precio
    initialValues.id_categoria = resValue.id_categoria
    initialValues.descripcion = resValue.descripcion
    initialValues.estado = resValue.estado
    src.value = resValue.imagen
    initialValues.foto = resValue.imagen
    const categoria = resValue.categoria

    const res:any[] = await $fetch(server.HOST + '/api/v1/categorias-productos', {
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
    nombre: z.string(),
    precio: z.number(),
    categoria: z.union([
      z.object({
        name: z.string().min(1, 'Categoria requerida.')
      }),
      z.any().refine((val) => true, { message: 'Categoria requerida.' })
    ]),
    foto: z.union([
      z.string(),
      z.instanceof(File, { message: 'Debe seleccionar una imagen.' })
        .refine((file) => file.type.startsWith('image/'), {
          message: 'El archivo debe ser una imagen válida.',
        })
        .refine((file) => file.size <= 2 * 1024 * 1024, {
          message: 'La imagen no debe superar los 2MB.',
        })
    ]).optional()
  })
))

const nombreError = ref('')

async function onFormSubmit({ valid } : any ) {
  if( valid ){
    const formData = new FormData()
    try{
      formData.append("nombre", initialValues.nombre)
      formData.append("precio", String(initialValues.precio))
      formData.append("descripcion", initialValues.descripcion)
      formData.append("id_categoria", String(initialValues.id_categoria))
      formData.append("estado", initialValues.estado)
      formData.append("foto", initialValues.foto as File)

      await $fetch(server.HOST + '/api/v1/productos/' + props.id, {
        method: 'PUT',
        body: formData
      })
      
      emit('update'), emit('success')
      visible.value = false
    } catch(err: any) {
      if (err.data && err.data.includes("Este nombre ya existe")) {
        nombreError.value = 'Este nombre ya existe'
      } else {
        emit('error')
      }
    }
  }
}

function onFileSelect(event: any) {
  const file = event.files[0]
  initialValues.foto = file

  const reader = new FileReader()
  reader.onload = (e) => {
    src.value = e.target?.result
  }
  reader.readAsDataURL(file)
}

</script>