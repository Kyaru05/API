<script setup>
import { ref, onMounted, computed } from 'vue'
import axios from 'axios'

// Lista de dólares que trae la API
const dolares = ref([])

// Estado de carga
const cargando = ref(true)
const error = ref('')

// Datos del <formulario
const tipoDolar = ref('')
const operacion = ref('comprar')
const cantidad = ref('')

// Obtener los dólares desde la API
async function obtenerDolares() {
  try {
    cargando.value = true

    const respuesta = await axios.get(
      'https://dolarapi.com/v1/dolares'
    )

    dolares.value = respuesta.data

    // Seleccionamos el primer dólar automáticamente
    if (dolares.value.length > 0) {
      tipoDolar.value = dolares.value[0].casa
    }

  } catch (e) {
    error.value = 'No se pudieron cargar las cotizaciones.'
  } finally {
    cargando.value = false
  }
}

// Buscar el dólar seleccionado
const dolarSeleccionado = computed(() => {
  return dolares.value.find(
    dolar => dolar.casa === tipoDolar.value
  )
})

// Calcular resultado
const resultado = computed(() => {
  if (!dolarSeleccionado.value || !cantidad.value) {
    return 0
  }

  const cantidadNumero = Number(cantidad.value)

  if (operacion.value === 'comprar') {
    // Pesos a dólares
    return cantidadNumero / dolarSeleccionado.value.venta
  } else {
    // Dólares a pesos
    return cantidadNumero * dolarSeleccionado.value.compra
  }
})

// Formatear números como pesos argentinos
function formato(numero) {
  return new Intl.NumberFormat('es-AR', {
    minimumFractionDigits: 2,
    maximumFractionDigits: 2
  }).format(numero)
}

// Cargar la API cuando inicia la página
onMounted(() => {
  obtenerDolares()
})
</script>

<template>
  <div class="fondo">

    <!-- Decoraciones -->
    <div class="decoracion decoracion1">♡</div>
    <div class="decoracion decoracion2">✦</div>
    <div class="decoracion decoracion3">♡</div>
    <div class="decoracion decoracion4">✿</div>
    <div class="decoracion decoracion5">✨</div>

    <main class="contenedor">

      <!-- TÍTULO -->
      <header class="titulo">
        <span>$</span>
        <div>
          <h1>Cotización del Dólar</h1>
          <p>Consultá los valores y calculá tu cambio ✨</p>
        </div>
        <span>$</span>
      </header>

      <!-- CARGANDO -->
      <div v-if="cargando" class="mensaje">
        <div class="spinner"></div>
        <p>Cargando cotizaciones...</p>
      </div>

      <!-- ERROR -->
      <div v-else-if="error" class="error">
        {{ error }}
        <button @click="obtenerDolares">
          Intentar nuevamente
        </button>
      </div>

      <div v-else>

        <!-- COTIZACIONES -->
        <section class="seccion">

          <h2>$ Tipos de dólar</h2>

          <div class="grid-dolares">

            <div
              v-for="dolar in dolares"
              :key="dolar.casa"
              class="tarjeta"
            >
              <div class="icono">$</div>

              <h3>{{ dolar.nombre }}</h3>

              <p class="actualizacion">
                Actualizado: {{ dolar.fechaActualizacion }}
              </p>

              <div class="precios">

                <div>
                  <span>Compra</span>
                  <strong>
                    $ {{ formato(dolar.compra) }}
                  </strong>
                </div>

                <div>
                  <span>Venta</span>
                  <strong>
                    $ {{ formato(dolar.venta) }}
                  </strong>
                </div>

              </div>
            </div>

          </div>

        </section>

        <!-- FORMULARIO -->
        <section class="calculadora">

          <h2>$ Compra y venta de dólares</h2>

          <p class="descripcion">
            Elegí el tipo de dólar y realizá tu cálculo.
          </p>

          <div class="formulario">

            <!-- TIPO DE DÓLAR -->
            <div class="campo">
              <label>Tipo de dólar</label>

              <select v-model="tipoDolar">

                <option
                  v-for="dolar in dolares"
                  :key="dolar.casa"
                  :value="dolar.casa"
                >
                  {{ dolar.nombre }}
                </option>

              </select>
            </div>

            <!-- OPERACIÓN -->
            <div class="campo">
              <label>Operación</label>

              <select v-model="operacion">

                <option value="comprar">
                  Comprar dólares
                </option>

                <option value="vender">
                  Vender dólares
                </option>

              </select>
            </div>

            <!-- CANTIDAD -->
            <div class="campo">
              <label>
                {{ operacion === 'comprar'
                  ? 'Pesos disponibles'
                  : 'Dólares disponibles' }}
              </label>

              <input
                v-model="cantidad"
                type="number"
                min="0"
                placeholder="Ingresá una cantidad"
              />
            </div>

          </div>

          <!-- RESULTADO -->
          <div
            v-if="cantidad && dolarSeleccionado"
            class="resultado"
          >

            <p>
              {{ operacion === 'comprar'
                ? 'Con tus pesos podés comprar:'
                : 'Por tus dólares recibirías:' }}
            </p>

            <h3>
              {{ operacion === 'comprar' ? 'USD' : '$' }}
              {{ formato(resultado) }}
            </h3>

            <small>
              Cotización:
              {{
                operacion === 'comprar'
                  ? 'Venta'
                  : 'Compra'
              }}
              ${{ formato(
                operacion === 'comprar'
                  ? dolarSeleccionado.venta
                  : dolarSeleccionado.compra
              ) }}
            </small>

          </div>

        </section>

        <!-- INFORMACIÓN -->
        <section class="info">

          <h3>💡 ¿Cómo funciona?</h3>

          <p>
            La aplicación obtiene las cotizaciones actuales
            desde una API y muestra los valores de compra y venta.
          </p>

          <p>
            Si elegís <strong>Comprar dólares</strong>,
            se toman tus pesos y se dividen por el precio de venta.
          </p>

          <p>
            Si elegís <strong>Vender dólares</strong>,
            se toman tus dólares y se multiplican por el precio de compra.
          </p>

        </section>

      </div>

      <footer>
        ✦ Trabajo práctico Vue 3 + Axios + API ✦
      </footer>

    </main>
  </div>
</template>

<style scoped>

* {
  box-sizing: border-box;
}

body {
  margin: 0;
}

/* FONDO */

.fondo {
  min-height: 100vh;
  padding: 50px 20px;
  background: linear-gradient(
    135deg,
    #fff0f6,
    #fce4ec,
    #f8bbd0,
    #fff5f8
  );

  position: relative;
  overflow: hidden;
  font-family: Arial, sans-serif;
}

/* DECORACIONES */

.decoracion {
  position: fixed;
  color: rgba(216, 27, 96, 0.20);
  font-size: 45px;
  animation: flotar 4s ease-in-out infinite;
}

.decoracion1 {
  top: 8%;
  left: 5%;
}

.decoracion2 {
  top: 20%;
  right: 8%;
  animation-delay: 1s;
}

.decoracion3 {
  bottom: 15%;
  left: 8%;
  animation-delay: 2s;
}

.decoracion4 {
  bottom: 8%;
  right: 10%;
  animation-delay: 1.5s;
}

.decoracion5 {
  top: 50%;
  left: 2%;
  animation-delay: 2.5s;
}

@keyframes flotar {

  0%, 100% {
    transform: translateY(0);
  }

  50% {
    transform: translateY(-15px);
  }

}

/* CONTENEDOR */

.contenedor {
  width: 100%;
  max-width: 1150px;
  margin: auto;
  position: relative;
  z-index: 2;
}

/* TÍTULO */

.titulo {
  background: rgba(255, 255, 255, 0.85);
  padding: 30px;
  border-radius: 25px;
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 25px;
  text-align: center;
  box-shadow: 0 10px 30px rgba(173, 20, 87, 0.15);
  margin-bottom: 30px;
}

.titulo span {
  font-size: 38px;
}

.titulo h1 {
  margin: 0;
  color: #c2185b;
  font-size: 38px;
}

.titulo p {
  margin-top: 10px;
  color: #777;
  font-size: 16px;
}

/* SECCIONES */

.seccion,
.calculadora,
.info {
  background: rgba(255, 255, 255, 0.90);
  padding: 30px;
  border-radius: 25px;
  box-shadow: 0 8px 25px rgba(173, 20, 87, 0.12);
  margin-bottom: 30px;
}

h2 {
  text-align: center;
  color: #c2185b;
  margin-top: 0;
}

/* GRID */

.grid-dolares {
  display: grid;
  grid-template-columns: repeat(
    auto-fit,
    minmax(230px, 1fr)
  );
  gap: 20px;
  margin-top: 25px;
}

/* TARJETAS */

.tarjeta {
  background: linear-gradient(
    145deg,
    #fff,
    #fff1f6
  );

  border: 2px solid #f8bbd0;
  border-radius: 20px;
  padding: 22px;
  text-align: center;

  transition: 0.3s;
}

.tarjeta:hover {
  transform: translateY(-7px);
  box-shadow: 0 10px 25px rgba(216, 27, 96, 0.18);
}

.icono {
  font-size: 35px;
}

.tarjeta h3 {
  color: #ad1457;
  margin: 10px 0;
}

.actualizacion {
  font-size: 11px;
  color: #999;
  min-height: 30px;
}

.precios {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 10px;
  margin-top: 20px;
}

.precios div {
  background: white;
  border-radius: 12px;
  padding: 12px;
}

.precios span {
  display: block;
  font-size: 13px;
  color: #888;
}

.precios strong {
  display: block;
  color: #c2185b;
  margin-top: 5px;
}

/* CALCULADORA */

.descripcion {
  text-align: center;
  color: #777;
}

.formulario {
  display: grid;
  grid-template-columns: repeat(
    auto-fit,
    minmax(220px, 1fr)
  );

  gap: 20px;
  margin-top: 25px;
}

.campo {
  display: flex;
  flex-direction: column;
}

.campo label {
  color: #ad1457;
  font-weight: bold;
  margin-bottom: 8px;
}

.campo input,
.campo select {
  padding: 14px;
  border-radius: 12px;
  border: 2px solid #f8bbd0;
  outline: none;
  background: white;
  font-size: 15px;
}

.campo input:focus,
.campo select:focus {
  border-color: #ec407a;
  box-shadow: 0 0 8px rgba(236, 64, 122, 0.20);
}

/* RESULTADO */

.resultado {
  margin-top: 30px;
  padding: 25px;
  text-align: center;
  border-radius: 20px;

  background: linear-gradient(
    135deg,
    #f8bbd0,
    #fce4ec
  );

  animation: aparecer 0.5s ease;
}

.resultado p {
  color: #880e4f;
}

.resultado h3 {
  color: #ad1457;
  font-size: 32px;
  margin: 10px;
}

.resultado small {
  color: #777;
}

@keyframes aparecer {

  from {
    opacity: 0;
    transform: translateY(10px);
  }

  to {
    opacity: 1;
    transform: translateY(0);
  }

}

/* INFORMACIÓN */

.info {
  text-align: center;
}

.info h3 {
  color: #c2185b;
}

.info p {
  color: #666;
  line-height: 1.7;
}

/* CARGANDO */

.mensaje {
  background: white;
  padding: 40px;
  border-radius: 20px;
  text-align: center;
  color: #c2185b;
}

.spinner {
  width: 40px;
  height: 40px;
  border: 4px solid #f8bbd0;
  border-top-color: #c2185b;
  border-radius: 50%;
  margin: auto;
  animation: girar 1s linear infinite;
}

@keyframes girar {
  to {
    transform: rotate(360deg);
  }
}

/* ERROR */

.error {
  background: #fff;
  padding: 30px;
  border-radius: 20px;
  text-align: center;
  color: #c2185b;
}

.error button {
  display: block;
  margin: 20px auto 0;
  padding: 12px 20px;
  border: none;
  border-radius: 12px;
  background: #c2185b;
  color: white;
  cursor: pointer;
}

/* FOOTER */

footer {
  text-align: center;
  color: #ad1457;
  padding: 20px;
  font-size: 14px;
}

/* CELULAR */

@media (max-width: 600px) {

  .titulo {
    padding: 20px;
    gap: 10px;
  }

  .titulo h1 {
    font-size: 26px;
  }

  .titulo span {
    font-size: 25px;
  }

  .seccion,
  .calculadora,
  .info {
    padding: 20px;
  }

}

</style>