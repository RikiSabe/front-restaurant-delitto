<template>
  <Drawer v-model:visible="visible" class="!w-full md:!w-80 lg:!w-[30rem]" position="right">
    <template #header>
      <span> <i class="pi pi-shopping-cart" /> Carrito </span>
    </template>
    <div class="mb-2">
      <Button 
        label="Limpiar todo el Carrito" 
        icon="pi pi-eraser" 
        size="small" 
        severity="danger"
        fluid 
        @click="limpiarCarrito" />
    </div>
    <DataTable
      :value="ProductosDelPedido"
      showGridlines
      size="small"
      editMode="cell"
      @cell-edit-complete="onCellEditComplete"
      :pt="{
        table: { style: 'min-width: 100%' },
        column: {
          bodycell: ({ state } : any) => ({
            class: [{ '!py-0': state['d_editing'] }]
          })
        }
      }" >
      <Column field="nombre" header="Nombre">
        <template #body="slotProps">
          <p class="text-xs text-center"> {{ slotProps.data.nombre }} </p>
        </template>
      </Column>
      <Column field="precio" header="Precio">
        <template #body="slotProps">
          <p class="text-xs text-center"> {{ slotProps.data.precio }} Bs </p>
        </template>
      </Column>
      <Column field="cantidad" header="#" style="width: 6rem">
        <template #body="{ data }">
          <p class="text-xs text-center">{{ data.cantidad }}</p>
        </template>
        <template #editor="{ data, field }">
          <InputNumber
            v-model="data[field]"
            inputClass="!text-xs !text-center"
            mode="decimal"
            :min="1" 
            :max="20"
            showButtons
            buttonLayout="horizontal"
            decrementButtonClass="p-button-outlined p-button-secondary"
            incrementButtonClass="p-button-outlined p-button-secondary"
            incrementButtonIcon="pi pi-plus"
            decrementButtonIcon="pi pi-minus" />
        </template>
      </Column>
      <Column>
        <template #body="slotProps">
          <div class="flex items-center justify-center gap-1">
            <Button
              variant="outlined"
              icon="pi pi-trash"
              size="small"
              severity="danger"
              @click="emit('remove', slotProps.data.id)" />
          </div>
        </template>
      </Column>
      <template #empty>
        <p class="text-center"> Sin Productos </p>
      </template>
      <template #footer>
        <div class="flex items-end justify-end">
          <p> Total: {{ totalPedido }} Bs </p>
        </div>
      </template>
    </DataTable>
    <div class="pt-2 pb-2">
      <Select
        :options="Mesas"
        optionLabel="nombre"
        optionValue="id"
        placeholder="Seleccionar Mesa"
        fluid
        v-model="seletedMesa" >
        <template #empty>
          <p class="text-center">
            No hay mesas disponibles, comuníquese con el mesero.
          </p>
        </template>
      </Select>
    </div>
    <template #footer>
      <Button v-if="showHacerPedido" label="Hacer pedido" fluid @click="realizarPedido" />
    </template>
  </Drawer>
</template>

<script setup lang="ts">
import { server } from '~/server/server'

interface Props {
  open: boolean
  modalValue: any
}

const props = defineProps<Props>()
const emit = defineEmits(['close', 'remove', 'clean', 'clean:ok', 'done'])

const visible = ref(props.open)
const ProductosDelPedido = toRef(props.modalValue)
const Mesas = ref<any[]>([])
const seletedMesa = ref<any>(null)

watch(visible, (newValue) => {
  if (!newValue) emit('close')
})

onMounted(async () => {
  try {
    const data: any[] = await $fetch(server.HOST + '/api/v1/mesas', { method: 'GET' })
    Mesas.value = data.filter((item: any) => item.estado === 'Disponible')
  } catch (err) {
    console.error(err)
  }
})

const totalPedido = computed(() =>
  ProductosDelPedido.value.reduce(
    (total: any, item: any) => total + item.precio * item.cantidad,
    0
  )
)

const limpiarCarrito = () => {
  emit('clean')
  emit('clean:ok')
  emit('close')
}

const onCellEditComplete = (event: any) => {
  const { data, newValue, field } = event

  if (field === 'cantidad') {
    if (isPositiveInteger(newValue)) {
      data[field] = newValue
    } else {
      event.preventDefault()
    }
  } else {
    event.preventDefault()
  }
}

const isPositiveInteger = (val: any) => {
  const n = Math.floor(Number(val))
  return n !== Infinity && String(n) === String(val) && n >= 1
}

const validations = computed(
  () => ProductosDelPedido.value.length > 0 && seletedMesa.value != null
)

const showHacerPedido = computed(() => validations.value)

const realizarPedido = async () => {
  try {
    await $fetch(server.HOST + '/api/v1/pedidos', {
      method: 'POST',
      body: {
        origen: 'panel-cliente',
        productos: JSON.parse(JSON.stringify(ProductosDelPedido.value)),
        estado: 'pendiente',
        id_mesa: seletedMesa.value
      }
    })
    ProductosDelPedido.value = []
    seletedMesa.value = null
    emit('done')
    emit('close')
    emit('clean')
  } catch (error) {
    console.error(error)
  }
}
</script>
