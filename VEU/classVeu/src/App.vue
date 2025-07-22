<template>
  <div class="calculadora">
    <div class="pantalla">
      {{ resultadoActual}}
    </div>

    <div class="botones">
       <button class="boton operador uno" @click="limpiarUno()">C</button>
      <button class="boton operador todo" @click="limpiarTodo()">AC</button>
      <button class="boton operador" @click="porcentaje()">%</button>
      <button class="boton operador" @click="multiplicar()">×</button>

      <button class="boton" v-for="numero in [7,8,9]" :key="numero" @click="agregarDigito(numero.toString())">{{numero}}</button>
      <button class="boton operador" @click="dividir()">÷</button>

      <button class="boton" v-for="numero in [4,5,6]" :key="numero" @click="agregarDigito(numero.toString())">{{numero}}</button>
      <button class="boton operador" @click="restar()">-</button>

      <button class="boton" v-for="numero in [1,2,3]" :key="numero" @click="agregarDigito(numero.toString())">{{numero}}</button>
      <button class="boton operador suma" @click="sumar()">+</button>

      <button class="boton" @click="agregarDigito('.')">.</button>
      <button class="boton" @click="agregarDigito('0')">0</button>
      <button class="boton igual" @click="calcularResultado()">=</button>
    </div>
  </div>
</template>

<script setup>
  import { ref } from 'vue';

  let ingresoActual = ref('');
  let ingresoAnterior = ref('');
  let operadorSeleccionado = ref(null);
  let resultadoActual = ref('0');

function agregarDigito(valor) {
  if (ingresoActual.value === '0' && valor !== '.') {
    ingresoActual.value = valor;
  }
  else if (valor === '.' && ingresoActual.value.includes('.')) {
    return;
  }
  else {
    ingresoActual.value += valor;
  }
  resultadoActual.value = ingresoActual.value;
}

function porcentaje() {
  if (ingresoActual.value !== '') {
    ingresoActual.value = (parseFloat(ingresoActual.value) / 100).toString();
    resultadoActual.value = ingresoActual.value;
  }
}

function limpiarUno() {
  ingresoActual.value = ''; 
  resultadoActual.value = '0'; 
}

function limpiarTodo() {
  ingresoActual.value = '';
  ingresoAnterior.value = '';
  operadorSeleccionado.value = null;
  resultadoActual.value = '0'; 
}

function establecerOperador(operador) {
  if (ingresoActual.value === '' && ingresoAnterior.value === '') return;

  if (ingresoActual.value !== '' && ingresoAnterior.value !== '' && operadorSeleccionado.value) {
    calcularResultado();
  }

  if (ingresoActual.value !== '') {
    ingresoAnterior.value = ingresoActual.value;
  }
  operadorSeleccionado.value = operador;
  ingresoActual.value = '';
}

function sumar() {
  establecerOperador('+');
}

function restar() {
  establecerOperador('-');
}

function multiplicar() {
  establecerOperador('*');
}

function dividir() {
  establecerOperador('/');
}

function calcularResultado() {
  let resultado;
  const numeroAnterior = parseFloat(ingresoAnterior.value);
  const numeroActual = parseFloat(ingresoActual.value || ingresoAnterior.value);

  if (isNaN(numeroAnterior) || isNaN(numeroActual)) {
    resultadoActual.value = '0'; 
    return;
  }

  switch (operadorSeleccionado.value) {
    case '+':
      resultado = numeroAnterior + numeroActual;
      break;
    case '-':
      resultado = numeroAnterior - numeroActual;
      break;
    case '*':
      resultado = numeroAnterior * numeroActual;
      break;
    case '/':
      
      if (numeroActual === 0) {
        resultadoActual.value = 'Error'; 
        ingresoActual.value = '';
        ingresoAnterior.value = '';
        operadorSeleccionado.value = null;
        return;
      }
      resultado = numeroAnterior / numeroActual;
      break;
    default:
      resultadoActual.value = ingresoActual.value || ingresoAnterior.value || '0';

      if (operadorSeleccionado.value === null && ingresoActual.value === '') {
        ingresoActual.value = resultadoActual.value;
      }
      return; 
  }
    ingresoActual.value = resultado.toString();
    resultadoActual.value = resultado.toString();
    operadorSeleccionado.value = null;
    ingresoAnterior.value = '';
}
</script>

<style scoped>

.calculadora {
  width: 400px;
  background-color: #333;
  border-radius: 10px;
  box-shadow: 0px 0px 20px rgba(0, 0, 0, 0.5);
  font-family: Arial, sans-serif;
  padding: 20px;
}

.pantalla {
  background-color: #ffffff;
  color: #000000;
  font-size: 45px;
  padding: 15px;
  margin-bottom: 15px;
  border-radius: 5px;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  word-wrap: break-word;
  word-break: break-all;
}

.botones {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  gap: 10px;
}

.boton {
  background-color: #555;
  color: #fff;
  border: none;
  padding: 20px;
  font-size: 1.5em;
  border-radius: 5px;
}

.boton:hover {
  background-color: #666;
}

.boton.operador:hover {
  background-color: #e6812a;
}

.boton.igual {
  background-color: #4CAF50;
}

.boton.igual:hover {
  background-color: #45a049;
}

.boton.suma {
  grid-row-end: span 2; 
}

.boton.uno {
  background-color: #e6812a;
}

.boton.todo {
  background-color: #e6812a;
}
</style>