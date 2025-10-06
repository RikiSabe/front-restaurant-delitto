<template>
  <Toast position="bottom-right" />
  <section class="flex justify-between p-4 m-4 ring ring-slate-200 rounded-lg sticky top-0 z-50 bg-white">
    <p class="font-bold p-2"> Productos </p>
    <Button icon="pi pi-shopping-cart" variant="outlined" @click="visibleCarrito = true"/>
  </section>
  <section class="grid grid-cols-4 gap-2 p-4">
    <Card 
      v-for="producto in Productos" 
      :key="producto.id"
      style="width: auto; overflow: hidden;">
      <template #header>
        <img :alt="producto.id" :src="producto.imagen" class="w-full h-44"/>
      </template>
      <template #content>
        <div class="flex flex-col gap-1">
          <span class="text-center font-bold"> 
            {{ producto.nombre }} 
          </span>
          <span class="text-center text-sm"> 
            {{ producto.descripcion }} 
          </span>
          <span class="text-end text-sm m-1 font-bold font-serif"> 
            {{ producto.precio }} bs 
          </span>
        </div>
      </template>
      <template #footer>
        <Button 
          label="Agregar al carrito"
          @click="agregarProducto(producto.id)" 
          size="small" fluid />
      </template>
    </Card>
  </section>

  <DetallesCarrito 
    :open="visibleCarrito"
    v-if="visibleCarrito"
    @close="visibleCarrito = false"
    @remove="eliminarProductoDelCarrito"
    @clean="Carrito = []"
    @clean:ok="toast.add({ severity: 'success', detail: 'Carrito Vacio :)', life:3000})"
    :modalValue="Carrito" />

</template>

<script setup lang="ts">
import DetallesCarrito from '~/components/cliente/carrito/detalles-carrito.vue'
import { server } from '~/server/server'

const visibleCarrito = ref(false)
const Productos = ref<any[]>([])
const Carrito = ref<any[]>([])
const toast = useToast()

onMounted( async () => {
  await obtenerProductos()
})

async function obtenerProductos(){
  try {
    Productos.value = await $fetch(server.HOST + '/api/v1/productos', {
      method: 'GET'
    })
  } catch(err){
    console.error(err)
  }
}

function agregarProducto(id: number) {
  const producto = Productos.value.find(p => p.id === id)
  if (!producto) return
  const index = Carrito.value.findIndex(item => item.id === id)
  if (index !== -1) {
    Carrito.value[index].cantidad += 1
    toast.add({
      severity:'info', 
      detail: `${Carrito.value[index].nombre} ya en carrito, se agregó +1 a la cantidad`, 
      life: 3000
    })
  } else {
    Carrito.value.push({
      ...producto,
      cantidad: 1
    })
    toast.add({
      severity:'success',
      detail: `${producto.nombre} agregado al carrito`,
      life: 3000
    })
  }
}

function eliminarProductoDelCarrito(id: number) {
  const index = Carrito.value.findIndex(p => p.id === id)
  if (index !== -1) {
    Carrito.value.splice(index, 1)
  }
}

</script>