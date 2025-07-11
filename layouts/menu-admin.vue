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
    <div class="p-2">
      <NuxtPage />
    </div>
  </div>
</template>

<script setup lang="ts">
const items = ref([
  {
    label: 'Gestion de Usuarios',
    items: [
      {
        label: 'Usuarios',
        route: 'usuarios'
      }
    ], 
  },
  {
    label: 'Gestion de Productos',
    items: [
      {
        label: 'Categorias',
        route: 'categorias'
      },
      {
        label: 'Productos',
        route: 'productos',
      },
      {
        label: 'Proveedores',
        route: 'proveedores',
      },
      {
        label: 'Insumos',
        route: 'insumos',
      }, 
    ]
  },
  {
    label : 'Gestion de Pedidos',
    items : [
      // {
      //   label: 'Mesas',
      //   route: 'mesas'
      // },
      {
        label: 'Pedidos',
        route: 'pedidos',
      },
    ]
  },
  {
    label : 'Sesion',
    items: [
      {
        label: 'Cerrar Sesión',
        route: '/'
      }
    ]
  }
]) 
</script>