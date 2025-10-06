<template>
  <div class="flex">
    <div class="w-56 h-dvh overflow-y-auto p-2">
      <Menu :model="items">
        <template #item="{item, props}">
          <router-link 
            v-if="item.route" v-slot="{ href, navigate }" :to="item.route" custom>
            <a v-ripple :href="href" v-bind="props.action" @click="navigate">
              <span>{{ item.label }}</span>
            </a>
          </router-link>
        </template>
      </Menu>
    </div>
    <div class="flex-1 p-2 overflow-y-auto">
      <NuxtPage />
    </div>
  </div>
</template>

<script setup lang="ts">
const items = ref([
  {
    label: 'Pedidos',
    items: [
      {
        label: 'Registrar Pedido',
        route: 'pedido'
      },
      {
        label: 'Historial',
        route: 'historial'
      },
      {
        label: 'Mesas',
        route: 'mesas'
      }
    ]
  },
  {
    label: 'Sesion',
    items: [
      {
        label: 'Cerrar Sesión',
        route: '/'
      }
    ]
  }
])
</script>