<template>
  <div class="flex flex-col h-dvh">
    <!-- HEADER -->
    <header class="flex items-center justify-between p-2 border-b bg-white">
      <div class="flex items-center gap-2 text-xl font-bold w-52">
        <i class="pi pi-th-large"></i>
        <span>Panel</span>
      </div>

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
        <div class="flex flex-col gap-2">
          <a @click="showGestionUsuarios = !showGestionUsuarios" class="block rounded-lg px-3 py-2 transition-colors select-none cursor-pointer" :class="showGestionUsuarios ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
            <span>Gestion de Usuarios</span>
          </a>
          <div v-if="showGestionUsuarios" class="flex flex-col gap-2 pl-4">
            <router-link to="/admin/usuarios" custom v-slot="{ href, navigate, isActive }">
              <a v-ripple :href="href" @click="navigate" class="block rounded-lg px-3 py-2 transition-colors select-none" :class="isActive ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
                <span>Usuarios</span>
              </a>
            </router-link>
          </div>
          <a @click="showGestionProductos = !showGestionProductos" class="block rounded-lg px-3 py-2 transition-colors select-none cursor-pointer" :class="showGestionProductos ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
            <span>Gestion de Productos</span>
          </a>
          <div v-if="showGestionProductos" class="flex flex-col gap-2 pl-4">
            <router-link to="/admin/categorias-menu" custom v-slot="{ href, navigate, isActive }">
              <a v-ripple :href="href" @click="navigate" class="block rounded-lg px-3 py-2 transition-colors select-none" :class="isActive ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
                <span>Categorias</span>
              </a>
            </router-link>
            <router-link to="/admin/productos" custom v-slot="{ href, navigate, isActive }">
              <a v-ripple :href="href" @click="navigate" class="block rounded-lg px-3 py-2 transition-colors select-none" :class="isActive ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
                <span>Productos</span>
              </a>
            </router-link>
          </div>
          <a @click="showGestionInsumos = !showGestionInsumos" class="block rounded-lg px-3 py-2 transition-colors select-none cursor-pointer" :class="showGestionInsumos ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
            <span>Gestion de Insumos</span>
          </a>
          <div v-if="showGestionInsumos" class="flex flex-col gap-2 pl-4">
            <router-link to="/admin/proveedores" custom v-slot="{ href, navigate, isActive }">
              <a v-ripple :href="href" @click="navigate" class="block rounded-lg px-3 py-2 transition-colors select-none" :class="isActive ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
                <span>Proveedores</span>
              </a>
            </router-link>
            <router-link to="/admin/categorias-insumos" custom v-slot="{ href, navigate, isActive }">
              <a v-ripple :href="href" @click="navigate" class="block rounded-lg px-3 py-2 transition-colors select-none" :class="isActive ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
                <span>Categorias</span>
              </a>
            </router-link>
            <router-link to="/admin/insumos" custom v-slot="{ href, navigate, isActive }">
              <a v-ripple :href="href" @click="navigate" class="block rounded-lg px-3 py-2 transition-colors select-none" :class="isActive ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
                <span>Insumos</span>
              </a>
            </router-link>
          </div>
          <a @click="showGestionPedidos = !showGestionPedidos" class="block rounded-lg px-3 py-2 transition-colors select-none cursor-pointer" :class="showGestionPedidos ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
            <span>Gestion de Pedidos</span>
          </a>
          <div v-if="showGestionPedidos" class="flex flex-col gap-2 pl-4">
            <router-link to="/admin/pedidos" custom v-slot="{ href, navigate, isActive }">
              <a v-ripple :href="href" @click="navigate" class="block rounded-lg px-3 py-2 transition-colors select-none" :class="isActive ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
                <span>Pedidos</span>
              </a>
            </router-link>
            <router-link to="/admin/mesas" custom v-slot="{ href, navigate, isActive }">
              <a v-ripple :href="href" @click="navigate" class="block rounded-lg px-3 py-2 transition-colors select-none" :class="isActive ? 'bg-yellow-500 text-white font-semibold' : 'text-gray-800 hover:bg-yellow-200'">
                <span>Mesas</span>
              </a>
            </router-link>
          </div>
        </div>
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

const showGestionUsuarios = ref(false)
const showGestionProductos = ref(false)
const showGestionInsumos = ref(false)
const showGestionPedidos = ref(false)

/* Usa PATHS reales (evitas el error de nombres como 'usuarios' vs 'admin-usuarios') */
const items = ref([
])
</script>
