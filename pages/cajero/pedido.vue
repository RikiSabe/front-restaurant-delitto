<template>
  <Toast />
  <div class="p-2 mb-2">
    <p class="text-xl font-bold">Realizar un pedido</p>
  </div>
  
  <div class="grid grid-cols-3 gap-2">
    
    <div class="col-span-2">
      <div class="grid grid-cols-3 gap-2">
        <Card 
          v-for="producto in Productos" 
          :key="producto.id"
          style="width: auto; overflow: hidden;">
          <template #header>
            <img :alt="producto.id" :src="producto.imagen" class="w-full h-44"/>
          </template>
          <template #content>
            <div class="flex flex-col gap-1">
              <span class="text-center"> {{ producto.nombre }} </span>
              <span class="text-xs"> {{ producto.descripcion }} </span>
              <span class="text-end text-xs"> {{ producto.precio }} bs </span>
            </div>
          </template>
          <template #footer>
            <Button 
              label="Agregar al pedido"
              @click="agregarProducto(producto.id)" 
              size="small" fluid />
          </template>
        </Card>
      </div>
    </div>

    <div class="col-span-1">
      <DataTable :value="ProductosDelPedido" show-gridlines size="small"> 
        <Column field="nombre" header="Nombre">
          <template #body="slotProps">
            <span class="text-xs"> {{ slotProps.data.nombre }} </span>
          </template>
        </Column>
        <Column field="precio" header="Precio">
          <template #body="slotProps">
            <span class="text-xs"> {{ slotProps.data.precio }} </span>
          </template>
        </Column>
        <Column field="cantidad" header="#">
          <template #body="slotProps">
            <span class="text-xs"> {{ slotProps.data.cantidad }} </span>
          </template>
        </Column>
        <Column>
          <template #body="slotProps">
            <div class="flex gap-1">
              <Button 
                variant="outlined" icon="pi pi-plus"
                size="small"
                @click="aumentarCantidad(slotProps.data.id)"/>
              <Button 
                variant="outlined" icon="pi pi-minus" 
                size="small"
                @click="disminuirCantidad(slotProps.data.id)"/>
            </div>
          </template>
        </Column>
        <template #empty>
          <span> Sin Productos </span>
        </template>
        <template #footer>
          <span text="text-end"> Total: {{ totalPedido }} </span>
        </template>
      </DataTable>

      <div class="mt-2 mb-2">
        <Select 
          :options="Mesas"
          v-model="id_mesa"
          placeholder="Seleccione una mesa"
          option-label="nombre" option-value="id" 
          fluid size="small" />
      </div>

      <Button 
        v-if="showHacerPedido"
        label="Realizar el pedido" 
        size="small" fluid
        @click="realizarPedido()"/>
    </div>
  </div>

</template>

<script setup lang="ts">
import { server } from '~/server/server'

definePageMeta({ layout : 'menu-cajero' })

const Productos = ref<any[]>([])
const Mesas = ref<any[]>([])
const id_mesa = ref(0)
const ProductosDelPedido = ref<any[]>([])
const toast = useToast()

onMounted( async () => {
  await ObtenerProductos()
  await ObtenerMesas()
})

const ObtenerProductos = async () => {
  try {
    const resValue: any[] = await $fetch(server.HOST + '/api/v1/mesas', {
      method: 'GET'
    })
    Mesas.value = resValue
  } catch (err) {
    console.error(err)
  }
}

const ObtenerMesas = async () => {
  try {
    const resValue: any[] = await $fetch(server.HOST + '/api/v1/productos', {
      method: 'GET'
    })
    Productos.value = resValue
  } catch( error ){
    console.error(error)
  }
}

const agregarProducto = (id: any) => {
  const producto = Productos.value.find(item => item.id === id)
  if (producto) {
    const existente = ProductosDelPedido.value.find(item => item.id === id)

    if (existente) {
      existente.cantidad++
    } else {
      ProductosDelPedido.value.push({
        id: producto.id,
        nombre: producto.nombre,
        precio: producto.precio,
        cantidad: 1
      })
    }
  }
}

const totalPedido = computed(() => 
  ProductosDelPedido.value.reduce((total, item) => {
    return total + (item.precio * item.cantidad)
  }, 0)
)

const aumentarCantidad = (id: any) => {
  const existente = ProductosDelPedido.value.find(item => item.id === id)
  if (existente) {
    existente.cantidad++
  }
}

const disminuirCantidad = (id: any) => {
  const existente = ProductosDelPedido.value.find(item => item.id === id)
  if (existente) {
    if (existente.cantidad > 1) {
      existente.cantidad--
    } else {
      ProductosDelPedido.value = ProductosDelPedido.value.filter(item => item.id !== id)
    }
  }
}

const validations = computed(() => {
  return ProductosDelPedido.value.length > 0 && id_mesa.value != 0
})

const showHacerPedido = computed(() => validations.value)

const realizarPedido = async () => {
  try {
    const response:any = await $fetch(server.HOST + '/api/v1/pedidos', {
      method: 'POST',
      body: {
        origen: 'caja',
        productos: JSON.parse(JSON.stringify(ProductosDelPedido.value)),
        estado: 'pendiente',
        id_mesa: id_mesa.value,
      }
    })
    ProductosDelPedido.value = []
    toast.add({ severity: 'success', summary: 'Pedido Realizado Exitosamente', life: 3000 })
    
    const factura = await fetch(server.HOST + '/api/v1/reportes/factura/' + response.id, {
      method: 'GET'
    })
    const blob = await factura.blob()
    const url = URL.createObjectURL(blob)
    window.open(url, '_blank')
  } catch (error) {
    console.error(error)
  }
}
</script>