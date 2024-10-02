<template>
  <LayoutAuthenticated>
    <div class="container mx-auto p-6">
      <h1 class="text-2xl font-bold mb-6">Lista de Habitaciones</h1>

      <!-- Botón para crear una nueva habitación -->
      <button
        @click="mostrarModalCrear"
        class="bg-green-500 text-white px-4 py-2 rounded-md mb-6 shadow-sm hover:bg-green-600 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-offset-2"
      >
        Crear Habitación
      </button>

      <!-- Modal para crear/editar una habitación -->
      <RoomModal
        :visible="showModal"
        :habitacion="habitacionSeleccionada"
        @close="cerrarModal"
        @save="guardarHabitacion"
      />

      <!-- Modal para mostrar detalles de la habitación -->
      <RoomDetailsModal
        :visible="showDetailsModal"
        :habitacion="habitacionDetalles"
        @close="showDetailsModal = false"
      />

      <!-- Modal para confirmar eliminación -->
      <CardBoxModal
        :modelValue="showConfirmModal"
        title="Confirmar Eliminación"
        button-label="Eliminar"
        hasCancel
        @confirm="eliminarHabitacionConfirmada"
        @cancel="showConfirmModal = false"
      >
        <p>¿Estás seguro de que deseas eliminar esta habitación?</p>
      </CardBoxModal>

      <!-- Tabla de habitaciones -->
      <table class="table-auto w-full border-collapse border border-gray-300 dark:border-gray-600">
        <thead>
          <tr class="bg-gray-200 dark:bg-gray-800">
            <th class="border border-gray-300 dark:border-gray-600 px-4 py-2">Número</th>
            <th class="border border-gray-300 dark:border-gray-600 px-4 py-2">Piso</th>
            <th class="border border-gray-300 dark:border-gray-600 px-4 py-2">Categoría</th>
            <th class="border border-gray-300 dark:border-gray-600 px-4 py-2">Estado</th>
            <th class="border border-gray-300 dark:border-gray-600 px-4 py-2">Precio</th>
            <th class="border border-gray-300 dark:border-gray-600 px-4 py-2">Características</th>
            <th class="border border-gray-300 dark:border-gray-600 px-4 py-2">Acciones</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="habitacion in habitaciones"
            :key="habitacion.id_habitacion"
            class="border-b border-gray-200 hover:bg-gray-100"
          >
            <td class="py-3 px-6 text-left">{{ habitacion.numero_habitacion }}</td>
            <td class="py-3 px-6 text-left">{{ habitacion.piso }}</td>
            <td class="py-3 px-6 text-left">{{ habitacion.categoria ? habitacion.categoria.tipo_habitacion: 'N/A' }}</td>
            <td class="py-3 px-6 text-left">
              <span :class="{
                  'inline-block px-3 py-1 text-xs font-semibold rounded-full': true,
                  'text-green-700 bg-green-200 dark:text-green-100 dark:bg-green-600': habitacion.estado === 'ACTIVO',
                  'text-yellow-700 bg-yellow-200 dark:text-yellow-100 dark:bg-yellow-600': habitacion.estado === 'MANTENIMIENTO',
                  'text-red-700 bg-red-200 dark:text-red-100 dark:bg-red-600': habitacion.estado === 'INACTIVO'
                }">
                {{ habitacion.estado }}
              </span>
            </td>
            <td class="py-3 px-6 text-left">{{ habitacion.precio_actual }}</td>
            <td class="py-3 px-6 text-left">
              <button
                @click="verDetalles(habitacion)"
                class="bg-yellow-500 text-white px-2 py-1 rounded-md ml-2 shadow-sm hover:bg-yellow-600 focus:outline-none focus:ring-2 focus:ring-yellow-500"
              >
                Detalles
              </button>
            </td>
            <td class="py-3 px-6 text-center">
              <button
                @click="editarHabitacion(habitacion)"
                class="bg-blue-500 text-white px-4 py-2 rounded-md mr-2 shadow-sm hover:bg-blue-600 focus:outline-none focus:ring-2 focus:ring-blue-500"
              >
                Editar
              </button>
              <button
                @click="confirmarEliminacion(habitacion.id_habitacion)"
                class="bg-red-500 text-white px-4 py-2 rounded-md shadow-sm hover:bg-red-600 focus:outline-none focus:ring-2 focus:ring-red-500"
              >
                Eliminar
              </button>
            </td>
          </tr>
        </tbody>
      </table>
    </div>
  </LayoutAuthenticated>
</template>

<script setup>
import { ref, onMounted } from 'vue';
import LayoutAuthenticated from '@/layouts/LayoutAuthenticated.vue';
import RoomModal from '@/components/juanca_components/RoomModal.vue';
import RoomDetailsModal from '@/components/juanca_components/RoomDetailsModal.vue';
import CardBoxModal from '@/components/CardBoxModal.vue';
import { obtenerTodasHabitaciones, eliminarHabitacion} from '@/services/juanca_service/habitacionService';

const habitaciones = ref([]);
const showModal = ref(false);
const showDetailsModal = ref(false);
const showConfirmModal = ref(false);
const habitacionSeleccionada = ref({});
const habitacionDetalles = ref({});
const habitacionAEliminar = ref(null);

const obtenerHabitaciones = async () => {
  try {
    const response = await obtenerTodasHabitaciones();
    habitaciones.value = response.data;
  } catch (error) {
    console.error("Error al obtener las habitaciones:", error.message);
  }
};

const verDetalles = (habitacion) => {
  habitacionDetalles.value = habitacion;
  showDetailsModal.value = true;
};

const eliminarHabitacionConfirmada = async () => {
  try {
    await eliminarHabitacion(habitacionAEliminar.value);
    showConfirmModal.value = false;
    habitacionAEliminar.value = null;
    obtenerHabitaciones();
  } catch (error) {
    console.error("Error al eliminar la habitación:", error.message);
  }
};

const confirmarEliminacion = (id_habitacion) => {
  habitacionAEliminar.value = id_habitacion;
  showConfirmModal.value = true;
};

const editarHabitacion = (habitacion) => {
  habitacionSeleccionada.value = habitacion;
  showModal.value = true;
};

const mostrarModalCrear = () => {
  habitacionSeleccionada.value = {
    id_habitacion: null,
    numero_habitacion: '',
    piso: '',
    id_categoria_habitacion: '',
    estado: '',
    precio_actual: '',
    caracteristicas: {},
  };
  showModal.value = true;
};

const guardarHabitacion = () => {
  // Llamar la función de API para guardar o actualizar habitación
  obtenerHabitaciones();
  cerrarModal();
};

const cerrarModal = () => {
  showModal.value = false;
  habitacionSeleccionada.value = {};
};

onMounted(() => {
  obtenerHabitaciones();
});
</script>
