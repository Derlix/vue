<template>
  <div v-if="visible" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50">
    <div class="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-lg max-w-2xl w-full relative">
      <button @click="cerrarModal"
        class="absolute top-4 right-4 text-gray-500 dark:text-gray-400 hover:text-gray-700 dark:hover:text-gray-300">
        &times;
      </button>
      <h2 class="text-xl font-semibold text-center mb-4">Crear Nuevo Huésped</h2>

      <div class="grid grid-cols-2 gap-4">
        <div>
          <label class="block text-sm font-medium text-gray-700 dark:text-gray-300">Nombre Completo</label>
          <input v-model="huesped.nombre_completo" type="text"
            class="w-full p-2 border border-gray-300 dark:border-gray-600 rounded-md bg-gray-50 dark:bg-gray-700 text-gray-900 dark:text-white"
            placeholder="Ingrese el nombre completo" required />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 dark:text-gray-300">Tipo de Documento</label>
          <select v-model="huesped.tipo_documento"
            class="w-full p-2 border border-gray-300 dark:border-gray-600 rounded-md bg-gray-50 dark:bg-gray-700 text-gray-900 dark:text-white"
            required>
            <option value="" disabled selected>Seleccione un tipo de documento</option>
            <option value="CC">CC</option>
            <option value="TI">TI</option>
            <option value="Pasaporte">PASAPORTE</option>
            <option value="Cedula_extranjera">CEDULA EXTRAJERA</option>
          </select>
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 dark:text-gray-300">Número de Documento</label>
          <input v-model="huesped.numero_documento" type="text"
            class="w-full p-2 border border-gray-300 dark:border-gray-600 rounded-md bg-gray-50 dark:bg-gray-700 text-gray-900 dark:text-white"
            placeholder="Ingrese número de documento" required />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 dark:text-gray-300">Fecha de Expedición</label>
          <input v-model="huesped.fecha_expedicion" type="date"
            class="w-full p-2 border border-gray-300 dark:border-gray-600 rounded-md bg-gray-50 dark:bg-gray-700 text-gray-900 dark:text-white"
            required />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 dark:text-gray-300">Email</label>
          <input v-model="huesped.email" type="email"
            class="w-full p-2 border border-gray-300 dark:border-gray-600 rounded-md bg-gray-50 dark:bg-gray-700 text-gray-900 dark:text-white"
            placeholder="Ingrese email" required />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 dark:text-gray-300">Teléfono</label>
          <input v-model="huesped.telefono" type="text"
            class="w-full p-2 border border-gray-300 dark:border-gray-600 rounded-md bg-gray-50 dark:bg-gray-700 text-gray-900 dark:text-white"
            placeholder="Ingrese teléfono" required />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 dark:text-gray-300">Ocupación</label>
          <input v-model="huesped.ocupacion" type="text"
            class="w-full p-2 border border-gray-300 dark:border-gray-600 rounded-md bg-gray-50 dark:bg-gray-700 text-gray-900 dark:text-white"
            placeholder="Ingrese ocupación" required />
        </div>

        <div>
          <label class="block text-sm font-medium text-gray-700 dark:text-gray-300">Dirección</label>
          <input v-model="huesped.direccion" type="text"
            class="w-full p-2 border border-gray-300 dark:border-gray-600 rounded-md bg-gray-50 dark:bg-gray-700 text-gray-900 dark:text-white"
            placeholder="Ingrese dirección" required />
        </div>
      </div>

      <button @click="crearhuesped"
        class="bg-blue-600 text-white py-2 px-4 rounded-md mt-4 hover:bg-blue-700 dark:bg-blue-500 dark:hover:bg-blue-600 transition">
        Crear Huésped
      </button>
    </div>
  </div>

    <!-- Modal de alerta -->
    <div v-if="alertVisible" class="fixed inset-0 flex items-center justify-center bg-black bg-opacity-50 z-50">
      <div class="bg-white dark:bg-gray-800 p-8 rounded-lg shadow-lg max-w-sm w-full">
        <h2 class="text-2xl font-semibold text-center mb-4 text-green-600 dark:text-green-400">
          Huésped creado con éxito
        </h2>
        <p class="text-center text-gray-600 dark:text-gray-300 mb-4">
          El nuevo huésped ha sido añadido a la base de datos.
        </p>
        <div class="flex justify-center">
          <button @click="cerrarAlerta" 
            class="bg-blue-600 text-white py-2 px-6 rounded-md hover:bg-blue-700 dark:bg-blue-500 dark:hover:bg-blue-600 transition">
            Aceptar
          </button>
        </div>
      </div>
    </div>
</template>

<script setup>
import { ref } from 'vue'
import { createHuesped } from '@/services/huespedService';

const props = defineProps({
  visible: Boolean
})

const emit = defineEmits(['close', 'crearHuesped'])

const huesped = ref({
  nombre_completo: '',
  tipo_documento: '',
  numero_documento: '',
  fecha_expedicion: '',
  email: '',
  telefono: '',
  ocupacion: '',
  direccion: ''
})

const alertVisible = ref(false);

const cerrarModal = () => {
  emit('close')
}

const cerrarAlerta = () => {
  alertVisible.value = false;
};

const crearhuesped = async () => {
  // Validar campos
  if (!huesped.value.nombre_completo || 
      !huesped.value.tipo_documento || 
      !huesped.value.numero_documento || 
      !huesped.value.fecha_expedicion || 
      !huesped.value.email || 
      !huesped.value.telefono || 
      !huesped.value.ocupacion || 
      !huesped.value.direccion) {
    console.error('Todos los campos son obligatorios');
    return;
  }

  // Convertir la fecha a formato YYYY-MM-DD
  huesped.value.fecha_expedicion = new Date(huesped.value.fecha_expedicion).toISOString().split('T')[0];

  try {
    console.log('Datos del nuevo huésped:', huesped.value);
    const response = await createHuesped(
      huesped.value.nombre_completo,
      huesped.value.tipo_documento,
      huesped.value.numero_documento,
      huesped.value.fecha_expedicion,
      huesped.value.email,
      huesped.value.telefono,
      huesped.value.ocupacion,
      huesped.value.direccion
    );
    console.log('Huésped creado con éxito:', response);
    
    // Cerrar el modal y mostrar la alerta
    cerrarModal();
    alertVisible.value = true; // Mostrar la alerta
  } catch (error) {
    if (error.response) {
      console.error('Detalles del error:', error.response.data);
    } else {
      console.error('Error de red o de servidor:', error);
    }
  }
};
</script>