<template>
  <Toast />
  <div class="p-2 mb-2">
    <p class="text-2xl" style="font-weight: 600;">Realizar un pedido</p>
  </div>
  
  <div class="grid grid-cols-1 lg:grid-cols-3 gap-4">
    
    <div class="lg:col-span-2">
      <div v-for="(productosCategoria, categoria) in productosAgrupados" :key="categoria" class="mb-8">
        <h2 class="category-title">{{ categoria }}</h2>
        <div class="grid grid-cols-1 sm:grid-cols-2 xl:grid-cols-3 gap-4 p-4">
          <Card 
            v-for="producto in productosCategoria" 
            :key="producto.id"
            class="product-card"
            style="width: auto; overflow: hidden;">
            <template #header>
              <!-- 
                Para una optimización completa con <picture>, el backend debería proporcionar 
                múltiples formatos (AVIF, WebP) y resoluciones (srcset) para cada imagen.
                Actualmente, solo usamos la imagen original.
              -->
              <picture>
                <!-- <source :srcset="producto.imagen_avif" type="image/avif"> -->
                <!-- <source :srcset="producto.imagen_webp" type="image/webp"> -->
                <img :src="producto.imagen" :alt="producto.nombre" class="w-full h-44 object-cover" loading="lazy"/>
              </picture>
            </template>
            <template #content>
              <div class="flex flex-col gap-2">
                <span class="product-name text-center"> {{ producto.nombre }} </span>
                <span class="product-description"> {{ producto.descripcion }} </span>
                <span class="product-price text-end"> {{ producto.precio }} bs </span>
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
    </div>

    <div class="col-span-1">
      <DataTable :value="ProductosDelPedido" show-gridlines size="small"> 
        <Column field="nombre" header="Nombre">
          <template #body="slotProps">
            <span class="text-sm"> {{ slotProps.data.nombre }} </span>
          </template>
        </Column>
        <Column field="precio" header="Precio">
          <template #body="slotProps">
            <span class="text-sm"> {{ slotProps.data.precio }} </span>
          </template>
        </Column>
        <Column field="cantidad" header="#">
          <template #body="slotProps">
            <InputNumber v-model="slotProps.data.cantidad" :min="1" :max="10" style="width: 5rem" inputClass="text-sm" />
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
          <span class="text-lg" style="font-weight: 500;"> Total: {{ totalPedido }} bs</span>
        </template>
      </DataTable>

      <div class="mt-4 mb-2">
        <Select 
          :options="Mesas"
          v-model="id_mesa"
          placeholder="Seleccione una mesa"
          option-label="nombre" option-value="id" 
          fluid size="small" />
      </div>

      <!-- <div class="mt-4 mb-2">
        <Textarea 
          v-model="comentario"
          placeholder="Escribir un comentario (opcional)"
          rows="3" fluid auto-resize />
      </div> -->

      <Button 
        v-if="showHacerPedido"
        label="Realizar el pedido" 
        size="large" fluid
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
const comentario = ref("")
const ProductosDelPedido = ref<any[]>([])
const toast = useToast()

onMounted( async () => {
  await ObtenerProductos()
  await ObtenerMesas()
})

const productosAgrupados = computed(() => {
  if (!Productos.value) return {}
  return Productos.value.reduce((acc, producto) => {
    const categoria = producto.categoria || 'Sin categoría'
    if (!acc[categoria]) {
      acc[categoria] = []
    }
    acc[categoria].push(producto)
    return acc
  }, {} as Record<string, any[]>)
})

const ObtenerMesas = async () => {
  try {
    const resValue: any[] = await $fetch(server.HOST + '/api/v1/mesas', {
      method: 'GET'
    })
    Mesas.value = resValue
  } catch (err) {
    console.error(err)
  }
}

const ObtenerProductos = async () => {
  try {
    const resValue: any[] = await $fetch(server.HOST + '/api/v1/productos', {
      method: 'GET'
    })
    Productos.value = resValue.filter(p => p.estado !== 'Inactivo')
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
  if (existente && existente.cantidad < 10) {
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
    // 1. Obtener el estado más reciente de la mesa seleccionada
    const mesaSeleccionada:any = await $fetch(`${server.HOST}/api/v1/mesas/${id_mesa.value}`);

    // 2. Verificar si la mesa está ocupada
    if (mesaSeleccionada && mesaSeleccionada.estado === 'Ocupado') {
      toast.add({ 
        severity: 'error', 
        summary: 'Mesa Ocupada', 
        detail: `La ${mesaSeleccionada.nombre} ya no está disponible.`, 
        life: 3000 
      });
      // Actualizar la lista de mesas en la UI para reflejar el estado real
      await ObtenerMesas();
      return; // Detener la ejecución
    }

    // 3. Si la mesa está disponible, proceder con el pedido
    const response:any = await $fetch(server.HOST + '/api/v1/pedidos', {
      method: 'POST',
      body: {
        origen: 'caja',
        productos: JSON.parse(JSON.stringify(ProductosDelPedido.value)),
        estado: 'pendiente',
        id_mesa: id_mesa.value,
        comentario: comentario.value
      }
    });

    ProductosDelPedido.value = [];
    toast.add({ severity: 'success', summary: 'Pedido Realizado Exitosamente', life: 3000 });
    
    const factura = await fetch(server.HOST + '/api/v1/reportes/factura/' + response.id, {
      method: 'GET'
    });

    const blob = await factura.blob();
    const url = URL.createObjectURL(blob);
    window.open(url, '_blank');

  } catch (error) {
    console.error(error);
    toast.add({ 
      severity: 'error', 
      summary: 'Error', 
      detail: 'No se pudo completar el pedido.', 
      life: 3000 
    });
  }
}
</script>