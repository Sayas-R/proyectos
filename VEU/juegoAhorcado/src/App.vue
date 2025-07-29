<template>

  <section id="section1" v-show="pagina === 'inicio'" class="inicio">
    <h1 class="titulo">El ahorcado</h1>

    <div class="secciones_inicio">
      <template v-if="!usuarioAutenticado">
        <div class="login">
        <h2>Iniciar o Registrarse</h2>
          <div class="inputs_login">
            <input type="text" id="nombreUsuario" v-model="nombreUsuarioInicio"
              placeholder="Ingrese su nombre de usuario">
            <input type="password" id="contrasena" v-model="contrasenaInicio" placeholder="Ingrese su contraseña"autocomplete="current-password">
          </div>
          <div class="btn_login">
            <button @click="iniciarSesion">Iniciar Sesión</button>
            <p v-if="mensajeInicioS" class="mensajeError">{{ mensajeInicioS }}</p>
          </div>
        </div>
      </template>
      <template v-else>
          <div class="informacionDelUsuario">
            <h2 style="color: bisque;">¡Hola, {{ nombreUsuario }}!</h2>
            <button class="btn_inicio" @click="pagina = 'dificultad'">Empezar a Jugar</button>
            <button class="btn_logout" @click="cerrarSesion">Cerrar Sesión</button>
          </div>
        </template>

      <div class="seleccion_personaje">
        <p>aqui va el personaje</p>
      </div>
    </div>
  </section>

  <section v-show="pagina === 'dificultad' && usuarioAutenticado" class="dificultad">
    <div class="opciones">
      <button @click="seleccionarNivel('facil')">Fácil</button>
      <button @click="seleccionarNivel('medio')">Medio</button>
      <button @click="seleccionarNivel('dificil')">Difícil</button>
    </div>
    <button @click="pagina = 'inicio'">Volver</button>
    <button @click="pagina = 'categorias'">Continuar</button>
  </section>

  <section v-show="pagina === 'categorias'" class="categorias">
    <div class="opciones">
      <button @click="seleccionarCategoria('animales')">Animales</button>
      <button @click="seleccionarCategoria('paises')">Países</button>
      <button @click="seleccionarCategoria('frutas')">Frutas</button>
      <button @click="seleccionarCategoria('colores')">Colores</button>
    </div>
    <button @click="pagina = 'dificultad'">Volver</button>
    <button @click="iniciarJuego(); pagina = 'juego'">Jugar</button>
  </section>

  <section v-show="pagina === 'juego'" class="juego">
    <p :class="['mensaje', { 'ganado': estadoJuego === 'ganado', 'perdido': estadoJuego === 'perdido' }]">{{ mensajeJuego }}</p>

    <div class="areaDeJuego">
        <div class="dibujo">
          <div v-if="erroresCometidos >= 1" class="horca"></div>
          <div v-if="erroresCometidos >= 2" class="soga"></div>
          <div v-if="erroresCometidos >= 3" class="cabeza"></div>
          <div v-if="erroresCometidos >= 4" class="baseCuerpo"></div>
          <div v-if="erroresCometidos >= 5" class="brazoDerecho"></div>
          <div v-if="erroresCometidos >= 6" class="brazoIzquierdo"></div>
          <div v-if="erroresCometidos >= 7" class="piernaDerecha"></div>
          <div v-if="erroresCometidos >= 8" class="piernaIzquierda"></div>
        </div>

        <h2 class="mostrar-palabra">{{ palabraMostrada.split('').join(' ') }}</h2>

        <p class="letras-usadas">
          Letras usadas:
          <span v-for="(letra, indice) in letrasIntentadas" :key="indice"
                :class="{ 'correcta': palabraSecreta.includes(letra), 'incorrecta': !palabraSecreta.includes(letra) }">
            {{ letra }}{{ indice < letrasIntentadas.length - 1 ? ', ' : '' }}
          </span>
        </p>

        <div class="seccion-estadisticas">
        <h2>Historial de Partidas de {{ nombreUsuario }}</h2>
        <p v-if="historialPartidas.length === 0">Aún no hay partidas registradas para este usuario.</p>
        <ul>
          <li v-for="(registro, indice) in historialPartidas" :key="indice" :class="{ 'registro-ganado': registro.resultado === 'ganado', 'registro-perdido': registro.resultado === 'perdido' }">
            <span>{{ registro.resultado === 'ganado' ? '🎉 Victoria' : '😞 Derrota' }}</span>
            <span>Categoría: {{ registro.categoria }}</span>
            <span>Nivel: {{ registro.nivel }}</span>
            <span>Palabra: {{ registro.palabra }}</span>
            <span>Errores: {{ registro.errores }}</span>
            <span>Fecha: {{ new Date(registro.fecha).toLocaleDateString('es-CO') }}</span>
          </li>
        </ul>
        <button v-if="historialPartidas.length > 0" @click="limpiarHistorial">Limpiar Historial</button>
      </div>
      </div>
    <!--<h2>{{ lineas }}</h2>
    <div>
      <button v-for="letra in 'abcdefghijklmnopqrstuvwxyz'" :key="letra" @click="adivinarLetra(letra)">
        {{ letra }}
      </button>
    </div>-->
  </section>
</template>

<script setup>
import { ref, computed,  watch, onMounted } from 'vue';

const pagina = ref('inicio');

const usuarioAutenticado = ref(false);
const nombreUsuarioInicio = ref('');
const contrasenaInicio = ref('');
const mensajeInicioS = ref('');

const palabras = {
  animales: ['perro', 'gato', 'pajaro', 'oso', 'anguila', 'caballo', 'cangrejo', 'vaca', 'toro', 'conejo'],
  paises: ['colombia', 'peru', 'panama', 'brazil', 'argentina', 'chile', 'mexico', 'españa', 'japon', 'italia'],
  frutas: ['manzana', 'pera', 'papaya', 'mora', 'banano', 'sandia', 'guanabana', 'fresa', 'mango'],
  colores: ['rojo', 'blanco', 'negro', 'azul', 'amarillo', 'verde', 'gris', 'naranja', 'morado', 'rosado']
};

const nivelSeleccionado = ref('');
const categoriaSeleccionada = ref('');
const palabraSecreta = ref('');
const letrasCorrectas = ref([]);

const lineas = computed(() => {
  if (palabraSecreta.value) {
    return palabraSecreta.value.split('').map(letra => letrasCorrectas.value.includes(letra) ? letra : '_').join(' ');
  } else {
    return '';
  }
});

function seleccionarNivel(nivel) {
  nivelSeleccionado.value = nivel;
}

function seleccionarCategoria(cat) {
  categoriaSeleccionada.value = cat;
}

function iniciarJuego() {
  if (!categoriaSeleccionada.value) {
    alert('Selecciona una categoría primero');
    return;
  }
  const lista = palabras[categoriaSeleccionada.value];
  palabraSecreta.value = lista[Math.floor(Math.random() * lista.length)];
  letrasCorrectas.value = [];
}

function adivinarLetra(letra) {
  if (palabraSecreta.value.includes(letra) && !letrasCorrectas.value.includes(letra)) {
    letrasCorrectas.value.push(letra);
  }
}
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Jacquard+12&display=swap');
@import url('https://fonts.googleapis.com/css2?family=Jacquard+12&family=Jacquard+12+Charted&display=swap');

html,
body {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
}

.inicio {
  background: url('inicio.gif') no-repeat center center fixed;
  background-size: cover;
  width: 100vw;
  height: 100vh;
  display: flex;
  flex-direction: column;
  align-items: center;
  text-align: center;
  color: #fff;
  font-family: "Jacquard 12", cursive;
}

.titulo {
  margin-block-start: 0;
  font-size: 13rem;
  margin-bottom: 5%;
}

.secciones_inicio {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 20px;
  padding: 20px;
  border-radius: 80px;
  width: 80%;
}

.login {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  width: 120%;
  height: 150%;
  background: rgba(0, 0, 0, 0.4);
  border-radius: 25px;
}

.inputs_login {
  display: flex;
  flex-direction: column;
  padding-bottom: 30px;
}

.inputs_login input {
  row-gap: 80px;
  border-radius: 3px;
  padding-bottom: 8px;
  color: antiquewhite;
  background-color: transparent;
}

.dificultad,
.categorias,
.juego {
  width: 100vw;
  height: 100vh;
  /* background-image: url("ruta/a/fondo.gif"); */
  background-size: cover;
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.opciones {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  width: 60%;
  margin: 20px auto;
}

.btn_inicio {
  background: #b30000;
  background-image: -webkit-linear-gradient(top, #b30000, #c95700);
  background-image: -moz-linear-gradient(top, #b30000, #c95700);
  background-image: -ms-linear-gradient(top, #b30000, #c95700);
  background-image: -o-linear-gradient(top, #b30000, #c95700);
  background-image: linear-gradient(to bottom, #b30000, #c95700);
  -webkit-border-radius: 6;
  -moz-border-radius: 6;
  border-radius: 6px;
  -webkit-box-shadow: 7px 8px 30px #fabe25;
  -moz-box-shadow: 7px 8px 30px #fabe25;
  box-shadow: 7px 8px 30px #fabe25;
  font-family: Courier New;
  color: #850000;
  font-size: 17px;
  padding: 8px 23px 6px 23px;
  border: dotted #d99f00 3px;
  text-decoration: none;
}

button:hover {
  background: #e30909;
  background-image: -webkit-linear-gradient(top, #e30909, #f06b05);
  background-image: -moz-linear-gradient(top, #e30909, #f06b05);
  background-image: -ms-linear-gradient(top, #e30909, #f06b05);
  background-image: -o-linear-gradient(top, #e30909, #f06b05);
  background-image: linear-gradient(to bottom, #e30909, #f06b05);
  text-decoration: none;
}

h1,
h2 {
  color: bisque;
  text-shadow: 2px 2px 0 rgba(0, 0, 0, 0.5);
}

.opciones div {
  text-align: center;
}

.opciones img {
  width: 100px;
  height: auto;
}
</style>