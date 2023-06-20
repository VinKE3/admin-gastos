<script setup>
import { ref, reactive, watch } from "vue";
import Prespuesto from "./components/Presupuesto.vue";
import ControlPresupuesto from "./components/ControlPresupuesto.vue";
import iconoNuevoGasto from "./assets/img/nuevo-gasto.svg";
import Modal from "./components/Modal.vue";
import Gasto from "./components/Gasto.vue";
import { generarId } from "./helpers";
const presupuesto = ref(0);
const disponible = ref(0);
const gastado = ref(0);
const gasto = reactive({
  nombre: "",
  cantidad: 0,
  categoria: "",
  id: null,
  fecha: Date.now(),
});

const gastos = ref([]);

const modal = reactive({
  mostrarModal: false,
  animarModal: false,
});

watch(
  gastos,
  () => {
    const totalGastado = gastos.value.reduce(
      (total, gasto) => gasto.cantidad + total,
      0
    );
    gastado.value = totalGastado;
    disponible.value = presupuesto.value - totalGastado;
  },
  {
    deep: true,
  }
);
watch(
  modal,
  () => {
    if (!modal.mostrarModal) {
      reiniciarFormulario();
    }
  },
  {
    deep: true,
  }
);

const mostrarModal = () => {
  modal.mostrarModal = true;
  setTimeout(() => {
    modal.animarModal = true;
  }, 300);
};

const cerrarModal = () => {
  setTimeout(() => {
    modal.mostrarModal = false;
  }, 300);
  modal.animarModal = false;
};

const definirPresupuesto = (cantidad) => {
  presupuesto.value = cantidad;
  disponible.value = cantidad;
};

const guardarGasto = () => {
  if (gasto.id) {
    //?editando
    const { id } = gasto;
    const indice = gastos.value.findIndex((gasto) => gasto.id === id);
    gastos.value[indice] = { ...gasto };
  } else {
    //?agregando
    gastos.value.push({ ...gasto, id: generarId() });
  }

  //quiero reiniciar el gasto
  reiniciarFormulario();
  cerrarModal();
};
const seleccionarGasto = (id) => {
  const gastoSeleccionado = gastos.value.find((gasto) => gasto.id === id);
  Object.assign(gasto, gastoSeleccionado);
  mostrarModal();
};

const reiniciarFormulario = () => {
  Object.assign(gasto, {
    nombre: "",
    cantidad: 0,
    categoria: "",
    id: null,
    fecha: Date.now(),
  });
};
const eliminarGasto = (id) => {
  const indice = gastos.value.findIndex((gasto) => gasto.id === id);
  if (confirm("¿Estás seguro de eliminar este gasto?")) {
    gastos.value.splice(indice, 1);
    cerrarModal();
  }
  console.log(id);
  console.log("eliminando");
};
</script>

<template>
  <div :class="{ fijar: modal.mostrarModal }">
    <header>
      <h1>Planificador de Gastos</h1>
      <div class="contenedor-header contenedor sombra">
        <Prespuesto
          v-if="presupuesto === 0"
          @definir-presupuesto="definirPresupuesto"
        />
        <ControlPresupuesto
          v-else
          :presupuesto="presupuesto"
          :disponible="disponible"
          :gastado="gastado"
        />
      </div>
    </header>
    <main v-if="presupuesto > 0">
      <div class="contenedor listado-gastos">
        <h2>
          {{ gastos.length > 0 ? "Listado de Gastos" : "No hay Gastos Aun" }}
        </h2>
        <Gasto
          v-for="gasto in gastos"
          :key="gasto.id"
          :gasto="gasto"
          :disponible="disponible"
          @seleccionar-gasto="seleccionarGasto"
        />
      </div>
      <div class="crear-gasto">
        <img :src="iconoNuevoGasto" alt="iconoNuevo" @click="mostrarModal" />
      </div>
      <Modal
        v-if="modal.mostrarModal"
        @cerrar-modal="cerrarModal"
        :modal="modal"
        v-model:nombre="gasto.nombre"
        v-model:cantidad="gasto.cantidad"
        v-model:categoria="gasto.categoria"
        @guardar-gasto="guardarGasto"
        @eliminar-gasto="eliminarGasto"
        :disponible="disponible"
        :id="gasto.id"
      />
    </main>
  </div>
</template>

<style>
:root {
  --azul: #3b82f6;
  --blanco: #ffffff;
  --gris-claro: #f5f5f5;
  --gris: #94a3b8;
  --gris-oscuro: #64748b;
  --negro: #000;
}
html {
  font-size: 62.5%;
  box-sizing: border-box;
}
*,
*:before,
*:after {
  box-sizing: inherit;
}
body {
  font-size: 1.6rem;
  font-family: "Lato", sans-serif;
  background-color: var(--gris-claro);
}
h1 {
  font-size: 4rem;
}
h2 {
  font-size: 3rem;
}
.fijar {
  overflow: hidden;
  height: 100vh;
}
header {
  background-color: var(--azul);
}
header h1 {
  padding: 3rem 0;
  margin: 0;
  color: var(--blanco);
  text-align: center;
}
.contenedor {
  width: 90%;
  max-width: 80rem;
  margin: 0 auto;
}
.contenedor-header {
  margin-top: -5rem;
  transform: translateY(5rem);
  padding: 5rem;
}
.sombra {
  box-shadow: 0px 10px 15px -3px rgba(0, 0, 0, 0.1);
  background-color: var(--blanco);
  border-radius: 1.2rem;
  padding: 5rem;
}
.crear-gasto {
  position: fixed;
  bottom: 5rem;
  right: 5rem;
}
.crear-gasto img {
  width: 5rem;
  cursor: pointer;
}
.listado-gastos {
  margin-top: 10rem;
}
.listado-gastos h2 {
  font-weight: 900;
  color: var(--gris-oscuro);
}
</style>
