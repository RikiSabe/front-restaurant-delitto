<template>
  <div class="flex h-dvh bg-gray-100">
    <!-- SIDEBAR -->
    <aside class="w-60 bg-indigo-900 text-white flex flex-col">
      <!-- Logo -->
      <div class="h-16 flex items-center justify-center gap-3 text-2xl border-b border-indigo-800">
        <span>&#128570;</span>
        <span class="font-semibold">NekoAdmin</span>
      </div>

      <!-- Menu -->
      <nav class="flex-1 p-2 space-y-2">
        <router-link to="/admin/usuarios" custom v-slot="{ href, navigate, isActive }">
          <a :href="href" @click="navigate" class="flex items-center gap-3 rounded-lg px-3 py-2 transition-colors" 
             :class="isActive ? 'bg-indigo-800' : 'hover:bg-indigo-700'">
            <i class="pi pi-users"></i>
            <span>Usuarios</span>
          </a>
        </router-link>
        <router-link to="/admin/categorias-menu" custom v-slot="{ href, navigate, isActive }">
          <a :href="href" @click="navigate" class="flex items-center gap-3 rounded-lg px-3 py-2 transition-colors" 
             :class="isActive ? 'bg-indigo-800' : 'hover:bg-indigo-700'">
            <i class="pi pi-tags"></i>
            <span>Categorias Menu</span>
          </a>
        </router-link>
        <router-link to="/admin/productos" custom v-slot="{ href, navigate, isActive }">
          <a :href="href" @click="navigate" class="flex items-center gap-3 rounded-lg px-3 py-2 transition-colors" 
             :class="isActive ? 'bg-indigo-800' : 'hover:bg-indigo-700'">
            <i class="pi pi-box"></i>
            <span>Productos</span>
          </a>
        </router-link>
        <router-link to="/admin/proveedores" custom v-slot="{ href, navigate, isActive }">
          <a :href="href" @click="navigate" class="flex items-center gap-3 rounded-lg px-3 py-2 transition-colors" 
             :class="isActive ? 'bg-indigo-800' : 'hover:bg-indigo-700'">
            <i class="pi pi-truck"></i>
            <span>Proveedores</span>
          </a>
        </router-link>
        <router-link to="/admin/categorias-insumos" custom v-slot="{ href, navigate, isActive }">
          <a :href="href" @click="navigate" class="flex items-center gap-3 rounded-lg px-3 py-2 transition-colors" 
             :class="isActive ? 'bg-indigo-800' : 'hover:bg-indigo-700'">
            <i class="pi pi-bookmark"></i>
            <span>Categorias Insumos</span>
          </a>
        </router-link>
        <router-link to="/admin/insumos" custom v-slot="{ href, navigate, isActive }">
          <a :href="href" @click="navigate" class="flex items-center gap-3 rounded-lg px-3 py-2 transition-colors" 
             :class="isActive ? 'bg-indigo-800' : 'hover:bg-indigo-700'">
            <i class="pi pi-shopping-cart"></i>
            <span>Insumos</span>
          </a>
        </router-link>
        <router-link to="/admin/pedidos" custom v-slot="{ href, navigate, isActive }">
          <a :href="href" @click="navigate" class="flex items-center gap-3 rounded-lg px-3 py-2 transition-colors" 
             :class="isActive ? 'bg-indigo-800' : 'hover:bg-indigo-700'">
            <i class="pi pi-book"></i>
            <span>Pedidos</span>
          </a>
        </router-link>
        <router-link to="/admin/mesas" custom v-slot="{ href, navigate, isActive }">
          <a :href="href" @click="navigate" class="flex items-center gap-3 rounded-lg px-3 py-2 transition-colors" 
             :class="isActive ? 'bg-indigo-800' : 'hover:bg-indigo-700'">
            <i class="pi pi-tablet"></i>
            <span>Mesas</span>
          </a>
        </router-link>
      </nav>
    </aside>

    <!-- Main Content -->
    <div class="flex-1 flex flex-col">
      <!-- HEADER -->
      <header class="h-16 flex items-center justify-between px-4 bg-gradient-to-r from-gray-900 to-gray-800 text-white sticky top-0 z-50 border-b border-white/20">
        <!-- Search Bar -->
        <div class="relative">
          <i class="pi pi-search absolute left-3 top-1/2 -translate-y-1/2 text-gray-400"></i>
          <input type="text" placeholder="Search..." class="pl-10 pr-4 py-2 rounded-lg border-transparent w-80 bg-white text-black">
        </div>
        
        <!-- User Info and Logout -->
        <client-only>
          <div class="flex items-center gap-4">
            <div class="flex items-center gap-2 cursor-pointer transition-opacity hover:opacity-80" @click="isProfileModalVisible = true">
                <i class="pi pi-user text-xl"></i>
                <div class="text-sm">
                    <div class="font-bold">{{ userData.nombre }}</div>
                    <div class="text-xs text-gray-300">{{ userData.rol }}</div>
                </div>
            </div>
            <NuxtLink
              to="/login"
              class="inline-flex items-center gap-2 rounded-lg px-3 py-2 transition-opacity hover:opacity-80"
            >
              <i class="pi pi-sign-out"></i>
              <span>Cerrar Sesión</span>
            </NuxtLink>
          </div>
        </client-only>
      </header>

      <!-- PAGE CONTENT -->
      <main class="flex-1 p-4 overflow-y-auto">
        <div class="bg-white p-4 rounded-lg border border-gray-200 shadow-sm">
          <NuxtPage />
        </div>
      </main>
    </div>

    <!-- User Profile Modal -->
    <UserProfileModal v-model:visible="isProfileModalVisible" :userData="userData" />
  </div>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { server } from '~/server/server';
import UserProfileModal from '~/components/shared/UserProfileModal.vue';

const userData = ref({
  id: 0,
  nombre: '',
  apellido: '',
  ci: '',
  celular: '',
  usuario: '',
  rol: '',
  estado: '',
});

const isProfileModalVisible = ref(false);

onMounted(async () => {
  const user = localStorage.getItem('user');
  if (user) {
    const parsedUser = JSON.parse(user);
    const userId = parsedUser.id;
    if (userId) {
      try {
        const res:any = await $fetch(server.HOST + '/api/v1/usuarios/' + userId, {
          method: 'GET'
        });
        Object.assign(userData.value, res);
      } catch (err) {
        console.error('Error fetching user data:', err);
      }
    }
  }
});
</script>
