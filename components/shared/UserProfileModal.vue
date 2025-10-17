<template>
  <Dialog :visible="visible" modal @update:visible="closeModal" :style="{ width: '28rem' }" :draggable="false">
    <template #header>
      <div class="flex flex-col items-center gap-3 w-full pt-4">
        <i class="pi pi-user text-6xl text-indigo-500 border-2 border-gray-300 rounded-full p-4"></i>
        <h2 class="text-2xl font-bold">{{ userData.nombre }} {{ userData.apellido }}</h2>
      </div>
    </template>

    <div class="flex flex-col gap-3 p-4 rounded-lg shadow-sm">
      <div class="grid grid-cols-[auto,1fr] items-center gap-x-4 gap-y-2 text-left">
        <span class="font-semibold py-1"><i class="pi pi-id-card mr-2"></i>CI:</span>
        <span class="py-1">{{ userData.ci }}</span>
        
        <span class="font-semibold py-1"><i class="pi pi-shield mr-2"></i>Rol:</span>
        <span class="px-2 py-1 text-sm rounded-md text-white" :class="userData.rol === 'admin' ? 'bg-indigo-500' : 'bg-blue-500'">{{ userData.rol }}</span>

        <span class="font-semibold py-1"><i class="pi pi-user mr-2"></i>Usuario:</span>
        <span class="py-1">{{ userData.usuario }}</span>

        <span class="font-semibold py-1"><i class="pi pi-phone mr-2"></i>Celular:</span>
        <span class="py-1">{{ userData.celular }}</span>
      </div>
    </div>
  </Dialog>
</template>

<script setup lang="ts">
import Dialog from 'primevue/dialog';
import Button from 'primevue/button'; // Button is no longer used, but keeping it for now

interface UserData {
  nombre: string;
  apellido: string;
  ci: string;
  rol: string;
  usuario: string;
  celular: string;
}

const props = defineProps<{
  visible: boolean;
  userData: UserData;
}>();

const emit = defineEmits(['update:visible']);

const closeModal = () => {
  emit('update:visible', false);
};
</script>
