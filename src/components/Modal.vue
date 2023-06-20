<script setup>
import cerrarModal from "../assets/img/cerrar.svg";
import { ref, computed } from "vue";
import Alerta from "./Alerta.vue";
const error = ref("");
const emit = defineEmits([
  "cerrar-modal",
  "update:nombre",
  "update:cantidad",
  "update:categoria",
  "guardar-gasto",
  "eliminar-gasto",
]);
const props = defineProps({
  modal: {
    type: Object,
    required: true,
  },
  nombre: {
    type: String,
    required: true,
  },
  cantidad: {
    type: [String, Number],
    required: true,
  },
  categoria: {
    type: String,
    required: true,
  },
  disponible: {
    type: Number,
    required: true,
  },
  id: {
    type: [String, null],
    required: true,
  },
});

const old = props.cantidad;

const isEditing = computed(() => {
  return props.id;
});

const agregarGasto = () => {
  const { nombre, cantidad, categoria, disponible, id } = props;
  if ([nombre, cantidad, categoria].includes("")) {
    error.value = "Todos los campos son obligatorios";
    setTimeout(() => {
      error.value = "";
    }, 3000);
    return;
  }
  //?validamos que la cantidad sea valida
  if (cantidad <= 0 || isNaN(cantidad)) {
    error.value = "Cantidad no valida";
    setTimeout(() => {
      error.value = "";
    }, 3000);
    return;
  }
  //?validamos que el usuario no gaste mas del presupuesto inicial
  if (id) {
    //?tomar en cuenta el gasto ya realizado
    if (cantidad > old + disponible) {
      error.value = "No puedes gastar mas de lo que tienes";
      setTimeout(() => {
        error.value = "";
      }, 3000);
      return;
    }
  } else {
    if (cantidad > disponible) {
      error.value = "No puedes gastar mas de lo que tienes";
      setTimeout(() => {
        error.value = "";
      }, 3000);
      return;
    }
  }
  emit("guardar-gasto");
};
</script>
<template>
  <div class="modal">
    <div class="cerrar-modal">
      <img :src="cerrarModal" @click="$emit('cerrar-modal')" />
    </div>
    <div
      class="contenedor contenedor-formulario"
      :class="[modal.animarModal ? 'animar' : 'cerrar']"
    >
      <form class="nuevo-gasto" @submit.prevent="agregarGasto">
        <legend>{{ isEditing ? "Editar Gasto" : "Nuevo Gasto" }}</legend>
        <Alerta v-if="error"> {{ error }}</Alerta>
        <div class="campo">
          <label for="nombre">Nombre Gasto:</label>
          <input
            type="text"
            id="nombre"
            placeholder="Añade el nombre del gasto"
            :value="nombre"
            @input="$emit('update:nombre', $event.target.value)"
          />
        </div>
        <div class="campo">
          <label for="cantidad">Cantidad:</label>
          <input
            type="number"
            id="cantidad"
            placeholder="Añade la cantidad del gasto"
            :value="cantidad"
            @input="$emit('update:cantidad', +$event.target.value)"
          />
        </div>
        <div class="campo">
          <label for="categoria">Categoria:</label>
          <select
            id="categoria"
            :value="categoria"
            @input="$emit('update:categoria', $event.target.value)"
          >
            <option value="">-- Seleccione Categoria --</option>
            <option value="ahorro">Ahorro</option>
            <option value="comida">Comida</option>
            <option value="casa">Casa</option>
            <option value="gastos">Gastos Varios</option>
            <option value="ocio">Ocio</option>
            <option value="salud">Salud</option>
            <option value="suscripciones">Suscripciones</option>
          </select>
        </div>
        <input
          type="submit"
          :value="isEditing ? 'Editar gasto' : 'Agregar gasto'"
        />
      </form>
      <button
        v-if="isEditing"
        type="button"
        class="btn-eliminar"
        @click="$emit('eliminar-gasto')"
      >
        Elminar Gasto
      </button>
    </div>
  </div>
</template>

<style lang="scss" scoped>
.modal {
  position: absolute;
  background-color: rgb(0 0 0 / 0.9);
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
}
.cerrar-modal {
  position: absolute;
  right: 3rem;
  top: 3rem;
}
.cerrar-modal img {
  width: 3rem;
  cursor: pointer;
}

.contenedor-formulario {
  transition-property: all;
  transition-duration: 0.3s;
  transition-timing-function: ease-in-out;
  opacity: 0;
}

.contenedor-formulario.animar {
  opacity: 1;
}

.contenedor-formulario.cerrar {
  opacity: 0;
}

.nuevo-gasto {
  margin: 10rem auto 0 auto;
  display: grid;
  gap: 2rem;
}

.nuevo-gasto label {
  font-size: 2.5rem;
  font-weight: bold;
  color: var(--blanco);
}

.nuevo-gasto legend {
  font-size: 2.5rem;
  font-weight: bold;
  text-transform: uppercase;
  color: var(--blanco);
  text-align: center;
}

.campo {
  display: grid;
  gap: 2rem;
}

.nuevo-gasto input,
.nuevo-gasto select {
  background-color: var(--gris-claro);
  border-radius: 1rem;
  padding: 1rem;
  border: none;
  font-size: 2.2rem;
}
.nuevo-gasto input[type="submit"] {
  background-color: var(--azul);
  color: #fff;
  font-size: 2.2rem;
  font-weight: bold;
  text-transform: uppercase;
  padding: 1rem;
  border-radius: 1rem;
  border: none;
  cursor: pointer;
}
.nuevo-gasto input[type="submit"]:hover {
  background-color: #2c3ab8;
}
.btn-eliminar {
  padding: 1rem;
  width: 100%;
  background-color: #ef4444;
  font-weight: 700;
  font-size: 2.3rem;
  color: var(--blanco);
  margin-top: 2rem;
  cursor: pointer;
  border-radius: 2rem;
  text-transform: uppercase;
}
.btn-eliminar:hover {
  background-color: #dc2626;
}
</style>
