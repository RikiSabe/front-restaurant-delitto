<template>
  <div class="flex flex-col h-dvh">
    <!-- HEADER -->
    <header class="flex items-center justify-between p-2 border-b bg-white">
      <div class="text-sm font-medium w-52">Panel</div>

      <!-- User Info (Centered) -->
      <div class="flex-1 flex justify-center items-center gap-4">
        <div class="flex items-center gap-2 bg-yellow-200 px-4 py-2 rounded-full">
            <i class="pi pi-user text-xl text-gray-600"></i>
            <div class="text-sm">
                <div class="font-bold text-gray-800">{{ userData.nombre }}</div>
                <div class="text-xs text-gray-500">{{ userData.rol }}</div>
            </div>
        </div>
      </div>

      <!-- Logout Button -->
      <div class="w-52 flex justify-end">
        <NuxtLink
          to="/login"
          class="inline-flex items-center gap-2 rounded px-3 py-2 transition-colors hover:bg-yellow-300 text-gray-900"
        >
          <i class="pi pi-sign-in"></i>
          <span>Cerrar Sesión</span>
        </NuxtLink>
      </div>
    </header>

    <!-- BODY -->
    <div class="flex flex-1">
      <!-- SIDEBAR -->
      <aside class="w-52 h-full overflow-y-auto p-2">
        <Menu :model="items">
          <template #item="{ item, props }">
            <router-link
              v-if="item.to"
              :to="item.to"
              custom
              v-slot="{ href, navigate, isActive }"
            >
              <a
                v-ripple
                :href="href"
                v-bind="props.action"
                @click="navigate"
                class="block rounded-lg px-3 py-2 transition-colors select-none"
                :class="isActive
                  ? 'bg-yellow-500 text-white font-semibold'
                  : 'text-gray-800 hover:bg-yellow-200'"
              >
                <span>{{ item.label }}</span>
              </a>
            </router-link>
          </template>
        </Menu>
      </aside>

      <main class="flex-1 p-2 overflow-y-auto">
        <NuxtPage />
      </main>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';

const userData = ref({
  nombre: '',
  rol: ''
});

onMounted(() => {
  const user = localStorage.getItem('user');
  if (user) {
    const parsedUser = JSON.parse(user);
    userData.value.nombre = parsedUser.nombre;
    userData.value.rol = parsedUser.rol;
  }
});

/* Usa PATHS reales (evitas el error de nombres como 'usuarios' vs 'admin-usuarios') */
const items = ref([
  {
    label: 'Gestion de Usuarios',
    items: [{ label: 'Usuarios', to: '/admin/usuarios' }],
  },
  {
    label: 'Gestion de Productos',
    items: [
      { label: 'Categorias', to: '/admin/categorias-menu' },
      { label: 'Productos',  to: '/admin/productos' },
    ]
  },
  {
    label: 'Gestion de Insumos',
    items: [
      { label: 'Proveedores', to: '/admin/proveedores' },
      { label: 'Categorias',  to: '/admin/categorias-insumos' },
      { label: 'Insumos',     to: '/admin/insumos' },
    ]
  },
  {
    label : 'Gestion de Pedidos',
    items : [
      { label: 'Pedidos', to: '/admin/pedidos' },
      { label: 'Mesas',   to: '/admin/mesas' },
    ]
  }
])
</script>
