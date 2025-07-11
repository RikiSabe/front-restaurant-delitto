<template>
  <Toast />
  <div class="p-2 mb-2">
    <p class="text-xl font-bold">Realizar un pedido</p>
  </div>

  <div class="flex gap-4">
    <div class="flex flex-col gap-2">
      <p class="text-xl font-bold">Selección de Productos</p>
      <Button label="Agregar Producto" @click="NuevoProductoCarrito = true"/>
      <DataTable :value="CarritoProductos" show-gridlines table-style="min-width: 34rem">
        <template #empty>
          <p class="text-center">Sin Productos seleccionados</p>
        </template>
        <template #footer>
          <p class="text-end"> Total : {{ totalProductos }} bs</p>
        </template>
        <Column field="id" header="ID" />
        <Column field="nombre" header="Nombre" />
        <Column field="precio" header="Precio">
          <template #body="slotProps">
            {{ slotProps.data.precio }} bs
          </template>
        </Column>
        <Column header="Acciones">
          <template #body="slotProps">
            <div class="flex items-center justify-center">
              <Button 
                icon="pi pi-trash" severity="danger" text 
                @click="EliminarProductoDelCarrito(slotProps.data)"/>
            </div>
          </template>
        </Column>
      </DataTable>
    </div>

    <div class="flex flex-col gap-2">
      <p class="text-xl font-bold">Selección de Insumos</p>
      <Button label="Agregar Insumo" @click="NuevoInsumoCarrito = true" />
      <DataTable :value="CarritoInsumos" show-gridlines table-style="min-width: 34rem">
        <template #empty>
          <p class="text-center">Sin Insumos seleccionados</p>
        </template>
        <Column field="id" header="ID" />
        <Column field="nombre" header="Nombre" />
        <Column field="cantidad" header="Cantidad" />
        <Column header="Acciones">
          <template #body="slotProps">
            <div class="flex items-center justify-center">
              <Button 
                icon="pi pi-trash" severity="danger" text 
                @click="EliminarInsumoDelCarrito(slotProps.data)"/>
            </div>
          </template>
        </Column>
      </DataTable>
    </div>
  </div>

  <div class="flex gap-4 mt-2 mb-2"> 
    <div>
      <p class="text-l font-bold">Seleccionar Mesa</p>
      <Select 
        placeholder="Seleccionar Mesa" :options="Mesas" 
        v-model="MesaSeleccionada"
        option-label="id" option-value="id"/>
    </div>

    <div>
      <p class="text-l font-bold">Seleccionar Usuario</p>
      <Select
        placeholder="Seleccionar Usuario" :options="Usuarios"
        v-model="UsuarioSeleccionado"
        option-label="nombre" option-value="id" />
    </div>
  </div>
  
  <Button class="mt-2" label="Hacer Pedido" @click="HacerRegistro" :disabled="!verificate"/>

  <Dialog v-model:visible="NuevoProductoCarrito" modal header="Agregar Producto al Carrito" :style="{width : '25rem'}" >
    <div class="flex flex-col gap-1">
      <label for="producto">Seleccione un producto </label>
      <Select
        :options="Productos" v-model="ProductoSeleccionado" 
        option-label="nombre" option-value="id"/>
    </div>
    <template #footer>
      <Button label="Agregar" fluid :disabled="!ProductoSeleccionado" @click="AgregarProductoAlCarrito(ProductoSeleccionado)"/>
    </template>
  </Dialog>

  <Dialog v-model:visible="NuevoInsumoCarrito" modal header="Agregar Insumo al Carrito" :style="{width : '25rem'}" >
    <div class="flex flex-col gap-1">
      <label for="insumo">Seleccione un insumo</label>
      <Select
        :options="Insumos" v-model="InsumoSeleccionado"
        option-label="nombre" option-value="id" />
    </div>
    <template #footer>
      <Button label="Agregar" fluid :disabled="!InsumoSeleccionado" @click="AgregarInsumoAlCarrito(InsumoSeleccionado)" />
    </template>
  </Dialog>
</template>

<script setup lang="ts">
import { server } from '~/server/server'

definePageMeta({ layout : 'menu-cajero' })

const toast = useToast()
const NuevoProductoCarrito = ref(false), NuevoInsumoCarrito = ref(false)
const CarritoInsumos = ref<any[]>([]), CarritoProductos = ref<any[]>([])
const Productos = ref<any[]>([]), Insumos = ref<any[]>([])
const ProductoSeleccionado = ref(), InsumoSeleccionado = ref()
const Mesas = ref<any[]>([])
const MesaSeleccionada = ref(0)
const Usuarios = ref<any[]>([])
const UsuarioSeleccionado = ref(0)

onMounted( async () => {
  try {
    const resProductos:any[] = await $fetch(server.HOST + '/api/v1/productos',{
      method: 'GET'
    })
    Productos.value = resProductos
    const resInsumos:any[] = await $fetch(server.HOST + '/api/v1/insumos', {
      method: 'GET'
    })
    Insumos.value = resInsumos
    const resUsuarios:any[] = await $fetch(server.HOST + '/api/v1/usuarios',{
      method: 'GET'
    })
    Usuarios.value = resUsuarios
    const resMesas:any[] = await $fetch(server.HOST + '/api/v1/mesas', {
      method: 'GET'
    })
    Mesas.value = resMesas
  } catch(err) {
    console.error(err)
  }
})

function AgregarProductoAlCarrito(idProducto: number) {
  const index = Productos.value.findIndex(p => p.id === idProducto)
  if (index !== -1) {
    const producto = Productos.value[index]
    CarritoProductos.value.push(producto)
    Productos.value.splice(index, 1)
    NuevoProductoCarrito.value = false
    ProductoSeleccionado.value = null
  }
}

function AgregarInsumoAlCarrito(idInsumo: number) {
  const index = Insumos.value.findIndex(i => i.id === idInsumo)
  if (index !== -1) {
    const insumo = Insumos.value[index]
    CarritoInsumos.value.push(insumo)
    Insumos.value.splice(index, 1)
    NuevoInsumoCarrito.value = false
    InsumoSeleccionado.value = null
  }
}

function EliminarProductoDelCarrito(producto: any) {
  const index = CarritoProductos.value.findIndex(p => p.id === producto.id)
  if (index !== -1) {
    CarritoProductos.value.splice(index, 1)
    Productos.value.push(producto)
  }
}

function EliminarInsumoDelCarrito(insumo: any) {
  const index = CarritoInsumos.value.findIndex(i => i.id === insumo.id)
  if (index !== -1) {
    CarritoInsumos.value.splice(index, 1)
    Insumos.value.push(insumo)
  }
}

async function HacerRegistro() {
  const idsProductos = CarritoProductos.value.map(p => p.id)
  const idsInsumos = CarritoInsumos.value.map(i => i.id)

  const pedido = {
    id_usuario: UsuarioSeleccionado.value,
    id_mesa: MesaSeleccionada.value,
    id_productos: idsProductos,
    id_insumos: idsInsumos
  }

  console.log("Pedido a enviar:", pedido)

  try {
    const response = await $fetch(server.HOST + '/api/v1/pedidos', {
      method: 'POST',
      body: pedido
    })
    console.log('Pedido registrado con éxito', response)
    CarritoProductos.value = []
    CarritoInsumos.value = []
    MesaSeleccionada.value = 0
    UsuarioSeleccionado.value = 0
    toast.add({ severity: 'success', summary: 'Pedido realizado con Exito', life: 3000})
  } catch (error) {
    console.error('Error al registrar el pedido:', error)
  }
}

const verificate = computed(() => {
  return (
    CarritoProductos.value.length > 0 &&
    CarritoInsumos.value.length > 0 &&
    MesaSeleccionada.value !== 0 &&
    UsuarioSeleccionado.value !== 0
  )
})

const totalProductos = computed(() => {
  const total = CarritoProductos.value.reduce((sum, producto) => sum + (producto.precio || 0), 0)
  return total.toFixed(2)
})

</script>