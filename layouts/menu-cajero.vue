<template>
  <div class="flex flex-col h-dvh">
    <!-- Barra superior -->
    <header class="flex items-center justify-between p-2 border-b">
      <div class="text-sm font-medium w-56">Cajero</div>

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
      <div class="w-56 flex justify-end">
        <router-link to="/" custom v-slot="{ href, navigate }">
          <a
            v-ripple
            :href="href"
            @click="navigate"
            class="inline-flex items-center gap-2 rounded px-3 py-2
                   bg-yellow-400 text-gray-900 hover:bg-yellow-300
                   transition-colors"
          >
            <i class="pi pi-sign-out"></i>
            <span>Cerrar Sesión</span>
          </a>
        </router-link>
      </div>
    </header>

    <!-- Cuerpo: menú + contenido -->
    <div class="flex flex-1">
      <div class="w-56 h-full overflow-y-auto p-2 border-r">
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
                class="block w-full rounded px-3 py-2 transition-colors
                       hover:bg-yellow-300 hover:text-gray-900"
                :class="isActive ? 'bg-yellow-300 text-gray-900' : ''"
              >
                <span>{{ item.label }}</span>
              </a>
            </router-link>

            <span v-else class="block px-3 py-2 text-sm text-gray-500">
              {{ item.label }}
            </span>
          </template>
        </Menu>
      </div>

      <div class="flex-1 p-2 overflow-y-auto">
        <NuxtPage />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';

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

const items = ref([
  {
    label: 'Pedidos',
    items: [
<<<<<<< HEAD
      { label: 'Registrar Pedido', to: '/cajero/pedido' },
      { label: 'Historial',        to: '/cajero/historial' },
      { label: 'Mesas',            to: '/cajero/mesas' }
=======
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
>>>>>>> 0fef626d350d081ed07e9a34011feffbf4246694
    ]
  }
  // Eliminamos el grupo "Sesion" del menú
])
</script>
