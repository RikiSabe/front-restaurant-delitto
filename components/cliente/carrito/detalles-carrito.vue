<template>
  <Drawer v-model:visible="visible" class="!w-full md:!w-80 lg:!w-[30rem]" position="right">
    <template #header>
      <span> <i class="pi pi-shopping-cart" /> Carrito </span>
    </template>

    <div class="mb-2">
      <Button label="Limpiar todo el Carrito" icon="pi pi-eraser" size="small" severity="danger"
        fluid @click="limpiarCarrito" />
    </div>
    <DataTable :value="ProductosDelPedido" show-gridlines size="small">
      <Column field="nombre" header="Nombre">
        <template #body="slotProps">
          <p class="text-xs text-center"> {{ slotProps.data.nombre }} </p>
        </template>
      </Column>
      <Column field="categoria" header="Categoría">
        <template #body="slotProps">
          <p class="text-xs text-center"> {{ slotProps.data.categoria }} </p>
        </template>
      </Column>
      <Column field="precio" header="Precio">
        <template #body="slotProps">
          <p class="text-xs text-center"> {{ slotProps.data.precio }} bs </p>
        </template>
      </Column>
      <Column field="cantidad" header="#">
        <template #body="slotProps">
          <InputNumber v-model="slotProps.data.cantidad" :min="1" :max="10" style="width: 5rem" inputClass="text-xs text-center" />
        </template>
      </Column>
      <Column>
        <template #body="slotProps">
          <div class="flex items-center justify-center gap-1">
            <Button variant="outlined" icon="pi pi-plus" size="small" @click="aumentarCantidad(slotProps.data.id)" />
            <Button variant="outlined" icon="pi pi-minus" size="small" @click="disminuirCantidad(slotProps.data.id)" />
          </div>
        </template>
      </Column>
      <template #empty>
        <p class="text-center"> Sin Productos </p>
      </template>
      <template #footer>
        <div class="flex items-end justify-end">
          <p> Total: {{ totalPedido }} bs. </p>
        </div>
      </template>
    </DataTable>
    <div class="pt-2 pb-2">
      <Select 
      :options="Mesas" optionLabel="nombre" optionValue="id"
      placeholder="Seleccionar Mesa" fluid
      v-model="seletedMesa">
      <template #empty>
        <p class="text-center"> No hay mesas disponibles, comuniquese con el mesero. </p>
      </template>
    </Select>
    </div>
    <template #footer>
      <Button  
        v-if="showHacerPedido" 
        label="Hacer pedido" fluid 
        @click="realizarPedido" />
    </template>
  </Drawer>
</template>

<script setup lang="ts">
import { server } from '~/server/server';

interface Props { open: boolean, modalValue: any }

const props = defineProps<Props>()
const emit = defineEmits(['close', 'remove', 'clean', 'clean:ok', 'done'])
const visible = ref(props.open)
const ProductosDelPedido = toRef(props.modalValue)
const Mesas = ref<any[]>([])
const seletedMesa = ref<any>(null)

watch(visible, (newValue) => {
  if (!newValue) { emit('close') }
})

onMounted( async() => {
  try {
    const data:any[] = await $fetch(server.HOST + '/api/v1/mesas', { 
      method: 'GET' 
    })
    Mesas.value = data.filter((item: any) => item.estado === "Disponible")
  } catch (err) {
    console.error(err)
  }
})

const totalPedido = computed(() =>
  ProductosDelPedido.value.reduce((total : any, item : any) => total + (item.precio * item.cantidad), 0)
)

const aumentarCantidad = (id: number) => {
  const existente = ProductosDelPedido.value.find((item:any) => item.id === id)
  if (existente && existente.cantidad < 10) {
    existente.cantidad++
  }
}

const disminuirCantidad = (id: number) => {
  const existente = ProductosDelPedido.value.find((item:any) => item.id === id)
  if (existente) {
    if (existente.cantidad > 1) {
      existente.cantidad--
    } else {
      emit('remove', id)
    }
  }
}

const limpiarCarrito = () => {
  emit('clean'), emit('clean:ok'), emit('close')
} 

const validations = computed(() => {
  return ProductosDelPedido.value.length > 0 && seletedMesa.value != null
})

const showHacerPedido = computed(() => validations.value)

const realizarPedido = async () => {
  console.log(ProductosDelPedido, seletedMesa)
  try {
    await $fetch(server.HOST + '/api/v1/pedidos', {
      method: 'POST',
      body: {
        origen: 'panel-cliente',
        productos: JSON.parse(JSON.stringify(ProductosDelPedido.value)),
        estado: 'pendiente',
        id_mesa: seletedMesa.value,
      }
    })
    ProductosDelPedido.value = []
    seletedMesa.value = null
    emit('done'), emit('close'), emit('clean')
  } catch (error) {
    console.error(error)
  }
}
</script>