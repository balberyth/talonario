<script setup>
import { ref } from "vue";
import Swal from "sweetalert2";

// Formulario
let premio = ref("");
let precioBoleta = ref("");
let tipoLoteria = ref("");
let fechaSorteo = ref("");

// FormularioBD
let nombreBD = ref("");
let telefonoBD = ref("");
let direccionBD = ref("");
let pagarBD = ref("");

let fsDate;
/* let fsAño = fsDate.getFullYear();
let fsMes = fsDate.getMonth();
let fsDia = fsDate.getDate(); */
let hoy = new Date();
hoy.setHours(0, 0, 0, 0);
/* let hAño = hoy.getFullYear();
let hMes = hoy.getMonth();
let hDia = hoy.getDate(); */

let mostrarFormulario = ref(true);

let mostrarInformacion = ref(false);

let mostrarFormularioBD = ref(false);


let boletaReservada = ref(false);
let boletaDisponible = ref(false);
let boletaComprada = ref(false);

let boletas = ref([]);

function generarLoteria() {
    fsDate = new Date(fechaSorteo.value + "T00:00:00");

    if (
        premio.value == "" &&
        precioBoleta.value == "" &&
        tipoLoteria.value == "" &&
        fechaSorteo.value == ""
    ) {
        Swal.fire({
            text: "Los datos están vacíos",
            icon: "error",
        });
    } else if (premio.value == "") {
        Swal.fire({
            text: "El premio está vacío",
            icon: "error",
        });
    } else if (isNaN(premio.value)) {
        Swal.fire({
            text: "Solo números en premio",
            icon: "warning",
        });
    } else if (precioBoleta.value == "") {
        Swal.fire({
            text: "El valor de la boleta está vacío",
            icon: "error",
        });
    } else if (isNaN(precioBoleta.value)) {
        Swal.fire({
            text: "Solo números en premio",
            icon: "warning",
        });
    } else if (tipoLoteria.value == "") {
        Swal.fire({
            text: "Seleccione un tipo de lotería",
            icon: "error",
        });
    } else if (fechaSorteo.value == "") {
        Swal.fire({
            text: "Ingrese la fecha final",
            icon: "error",
        });
    } else if (fsDate <= hoy) {
        Swal.fire({
            text: "Ingrese una fecha final válida",
            icon: "warning",
        });
    } else {
        mostrarInformacion.value = true;
        Swal.fire({
            text: "¡Gracias por usar este Talonario!",
            icon: "success",
        });
        if(boletas.value.length == 0) {
          for (let i = 0; i < 100; i++) {
            boletas.value.push({
                i,
                estado: "Disponible",
            });
        }
        }
        
        mostrarFormulario.value = false;

        console.log(boletas.value);
    }
}

function editar() {
    mostrarFormulario.value = true;
    mostrarInformacion.value = false;
}

let element = ref(null);
let index = ref(null);

function modificarBoleta(elemento, indice) {
    console.log(elemento)
    if (elemento.estado == "Disponible") {
        // mostrar desplegable de boletas disponible
        boletaDisponible.value = false;
        boletaDisponible.value = true;
        // esconde demas desplegables
        boletaReservada.value = false;
        boletaComprada.value = false;
        
        element = elemento;
        index = indice;
    } else if (elemento.estado == "Reservado") {
        // mostrar desplegable de boletas reservada
        boletaReservada.value = false;
        boletaReservada.value = true;
        // esconde demas desplegables
        boletaDisponible.value = false;
        boletaComprada.value = false;

        element = elemento;
        index = indice;
    } else {
        // mostrar desplegable de boletas comprada
        boletaComprada.value = false;
        boletaComprada.value = true;
        // esconde demas desplegables
        boletaDisponible.value = false;
        boletaReservada.value = false;

        element = elemento;
        index = indice;
    }
}

function mostrarFBD() {
    mostrarFormularioBD.value = false;
    mostrarFormularioBD.value = true;
}

let clientes = ref([]);

function adquirir() {
    if (
        nombreBD.value == "" &&
        telefonoBD.value == "" &&
        direccionBD.value == "" &&
        pagarBD.value == ""
    ) {
        Swal.fire({
            text: "Los datos están vacíos",
            icon: "error",
        });
    } else if (nombreBD.value == "") {
        Swal.fire({
            text: "El nombre está vacío",
            icon: "error",
        });
    } else if (telefonoBD.value == "") {
        Swal.fire({
            text: "El telefono está vacío",
            icon: "error",
        });
    } else if (isNaN(telefonoBD.value)) {
        Swal.fire({
            text: "Solo números en telefono",
            icon: "warning",
        });
    } else if (direccionBD.value == "") {
        Swal.fire({
            text: "La dirección está vacía",
            icon: "error",
        });
    } else if (pagarBD.value == "") {
        Swal.fire({
            text: "Seleccione si paga ahora o no",
            icon: "error",
        });
    } else if (pagarBD.value == "si") {
        Swal.fire({
            text: "¡Gracias por comprar la boleta!",
            icon: "success",
        });
        clientes.value.push({
            nombre: nombreBD.value,
            telefono: telefonoBD.value,
            direccion: direccionBD.value,
            pagar: pagarBD.value,
            indice: index,
        });
        boletas.value[index].estado = 'Comprado'
        mostrarFormularioBD.value = false;
        boletaDisponible.value = false;
    } else if (pagarBD.value == "no") {
        Swal.fire({
            text: "¡Gracias por reservar una boleta!",
            icon: "success",
        });
        clientes.value.push({
            nombre: nombreBD.value,
            telefono: telefonoBD.value,
            direccion: direccionBD.value,
            pagar: pagarBD.value,
            indice: index,
        });
        boletas.value[index].estado = 'Reservado'
        mostrarFormularioBD.value = false;
        boletaDisponible.value = false;
    }
}

function cambiarColorBoton(objeto) {
    switch (objeto) {
        case "Disponible":
            return { backgroundColor: "beige", color: "black" };
        case "Reservado":
            return { backgroundColor: "red" };
        case "Comprado":
            return { backgroundColor: "blue" };
    }
}
</script>

<template>
    <div id="all">
        <div id="header">
            <h1>TALONARIO</h1>
        </div>
        <div id="footer">
            <footer>©️ Copyright 2024 - Todos los derechos reservados</footer>
        </div>
        <!-- Desplegable boleta reservada -->
        <div id="boletaDesplegableReserva" v-if="boletaReservada == true">
            <h3>
                Boleta
                <span>{{ element.i > 10 ? element.i : "0" + element.i }}</span>
            </h3>
            <p>
                Estado:
                <span>Reservada 🔴</span>
            </p>
            <div>
                <button>Ver datos del participante</button>
                <button>Liberar boleta</button>
                <button>Marcar como pagada</button>
            </div>
        </div>
        <!-- Desplegable boleta comprada -->
        <div id="boletaDesplegableReserva" v-if="boletaComprada == true">
            <h3>
                Boleta
                <span>{{ element.i > 10 ? element.i : "0" + element.i }}</span>
            </h3>
            <p>
                Estado:
                <span>Comprada 🔵</span>
            </p>
            <div>
                <button>Ver datos del participante</button>
                <button>Liberar boleta</button>
            </div>
        </div>
        <!-- Desplegable boleta disponible -->
        <div id="boletaDesplegableDisponible" v-if="boletaDisponible == true">
            <h3>
                Boleta
                <span>{{ element.i > 10 ? element.i : "0" + element.i }}</span>
            </h3>
            <p>Estado: <span>Disponible ⚫</span></p>
            <div>
                <button @click="mostrarFBD()">🤝 Adquirir Boleta</button>
            </div>
        </div>
        <main>
            <section id="informacion">
                <h2>INFORMACIÓN</h2>
                <div id="infoContenido">
                    <p>
                        🏆<span>{{ mostrarInformacion == true ? premio : "" }}</span>
                    </p>
                    <p>
                        💰<span>{{ mostrarInformacion == true ? precioBoleta : "" }}</span>
                    </p>
                    <p>
                        🏦<span>{{ mostrarInformacion == true ? tipoLoteria : "" }}</span>
                    </p>
                    <p>
                        🗓️<span>{{ mostrarInformacion == true ? fechaSorteo : "" }}</span>
                    </p>
                    <button @click="editar()">Editar ✏️</button>
                </div>
            </section>
            <section id="loteria">
                <button v-for="(e, i) in  boletas " :key="i" @click="modificarBoleta(e, i)"
                    :style="cambiarColorBoton(e.estado)">
                    {{ e.i < 10 ? "0" + e.i : e.i }} </button>
            </section>
            <section id="accion">
                <h2>ACCIONES</h2>
                <div id="accContenido">
                    <button id="estadoB">ESTADO</button>
                    <button>LISTAR BOLETAS</button>
                    <button>PERSONALIZAR TALONARIO WEB</button>
                    <button>GENERAR ARCHIVO DE DATOS</button>
                </div>
            </section>
            <div id="fondo" v-if="mostrarFormulario == true"></div>
            <section id="formularioBD" v-if="mostrarFormularioBD == true">
                <h2>DILIGENCIA LA INFORMACION</h2>
                <p>
                    Boleta a Adquirir:
                    <span>{{ index <= 9 ? "0" + index : index }}</span>
                </p>
                <input type="text" id="dBD" placeholder="🧍🏻Nombre" v-model="nombreBD" />
                <input type="tel" placeholder="📞 Telefono" v-model="telefonoBD" />
                <input type="text" placeholder=" Direccion" v-model="direccionBD" />
                <p>Pagar ya:</p>
                <div>
                    <label for="pagarSi">
                        <input type="radio" name="pagar" id="pagarSi" value="si" v-model="pagarBD" />
                        Si
                    </label>
                    <label for="pagarSi">
                        <input type="radio" name="pagar" id="pagarNo" value="no" v-model="pagarBD" />
                        No
                    </label>
                </div>
                <button @click="adquirir()">ADQUIRIR</button>
            </section>
            <section id="formulario" v-if="mostrarFormulario == true">
                <h2>CONFIGURA TU TALONARIO</h2>
                <div id="forContenido">
                    <input type="text" placeholder="Premio" v-model="premio" />
                    <input type="text" placeholder="Valor Boleta" v-model="precioBoleta" />
                    <select v-model="tipoLoteria">
                        <option disabled value="">Selecciona una lotería</option>
                        <option value="Baloto" label="Baloto"></option>
                        <option value="La Culona" label="La Culona"></option>
                        <option value="Lotería de Bogotá" label="Lotería de Bogotá"></option>
                        <option value="Lotería de Boyacá" label="Lotería de Boyacá"></option>
                        <option value="Lotería del Huila" label="Lotería del Huila"></option>
                        <option value="Lotería de Medellín" label="Lotería de Medellín"></option>
                        <option value="Lotería del Tolima" label="Lotería del Tolima"></option>
                        <option value="Lotería del Valle" label="Lotería del Valle"></option>
                        <option value="Super Astro" label="Super Astro"></option>
                    </select>
                    <input type="date" id="editDate" v-model="fechaSorteo" />
                    <button @click="generarLoteria()">GUARDAR</button>
                </div>
            </section>
        </main>
    </div>
</template>

<style scoped>
body {
  width: 100% !important;
    background: linear-gradient(to bottom, #fad02e, #64b5f6) !important;
    font-family: 'Arial', sans-serif;
    margin: 0;
}

#header {
    position: fixed;
    top: 0;
    left: 50%;
    transform: translate(-50%);
    background-color: #3498db;
    color: #fff;
    font-size: 10px;
    padding: 5px 0 5px 0;
    width: 100%;
    box-shadow: 0 5px 10px rgba(0, 0, 0, 0.26);
}

#footer {
    position: fixed;
    bottom: 0;
    left: 50%;
    transform: translate(-50%);
    background-color: #3498db;
    color: #fff;
    padding: 5px 0 5px 0;
    width: 100%;
}

main {
    padding: 90px 0 0 0;
    display: flex;
    flex-direction: row;
    justify-content: space-around;

}

#informacion {
    margin-top: 10rem;
    color: black;
    background-color:  #3498db;
    border-radius: 20px;
    padding: 20px;
    width: 400px;
    height: 400px;
}

#informacion h2 {
    font-weight: 400;
    font-size: 20px;
}

#infoContenido {
    text-align: start;
    background-color: beige;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    padding: 40px;
    border-radius: 5px;
    
}
#infoContenido span {
    color: #333; /* Cambia este color al que desees */
}


#informacion button {
  background-color: #2ecc71; /* Verde */
    font-size: 14px;
    font-weight: 400;
    padding: 7px 10px;
    color: #fff;
    border: none;
    border-radius: 4px;
    cursor: pointer;

}

#accion {
    margin-top: 9rem;
    color: #000000;
    background-color: #3498db;
    border-radius: 20px;
    padding: 30px;
    width: 400px;
    height: 400px;
}

#accion h2 {
    font-weight: 400;
    font-size: 20px;
}

#accContenido {
  text-align: center;
    background-color: beige;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    padding: 50px;
    border-radius: 5px;
    font-size: 12px;
}

#accion button {
  width: 140px;
    margin-bottom: 5px;
    border-radius: 6px;
    background-color: #2ecc71; /* Verde */
    color: #fff;
    border: none;
    cursor: pointer;
}

#accion #estadoB {
    margin-top: 21px;
    margin-bottom: 10px;
    background-color: blueviolet;
    border-radius: 2px;
}

#fondo {
    position: absolute;
    background-color: #95a5a656;
    width: 100%;
    height: 100%;
    top: 0;
    z-index: -1;
  }

#formularioBD {
    display: flex;
    flex-direction: column;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    align-items: center;
    background-color: aliceblue;
    border: 1px solid black;
    color: #000;
    padding: 0 0 15px 0;
}

#formularioBD h2 {
    background-color: blue;
    color: #fff;
    padding: 5px 40px;
    font-size: 15px;
    position: relative;
    bottom: 12.2px;
}

#dBD,
#formularioBD input[type="tel"] {
    width: 202px;
    padding: 5px 10px 5px 10px;
    background-color: white;
    border: 1px solid gray;
    color: black;
    margin-bottom: 20px;
}

#formularioBD input[type="text"] {
    width: 202px;
    padding: 5px 10px 5px 10px;
    background-color: white;
    border: 1px solid gray;
    color: black;
}

#formularioBD div {
    display: flex;
    flex-direction: row;
    gap: 20px;
    margin-bottom: 15px;
}

#formularioBD button {
    background-color: blue;
    font-size: 13px;
    width: 100px;
    padding: 5px 10px 5px 10px;
    border-radius: 0;
}

#formulario {
  margin-top: 10rem;
    color: black;
    background-color:  #3498db;
    border-radius: 20px;
    padding: 20px;
    width: 240px;
    height: 350px;
    position: absolute;
  padding-left: 25px;
}

#formulario h2 {
  
    padding: 5px 40px;
    position: relative;
    bottom: 12.2px;
    font-weight: 400;
    font-size: 20px
}

#forContenido {
  text-align: start;
    background-color: beige;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    padding: 5px;
    padding-left: 8px;
    border-radius: 5px;
}

#forContenido input[type="text"] {
    width: 202px;
    padding: 5px 10px 5px 10px;
    background-color: white;
    border: 1px solid gray;
    color: black;
    margin-bottom: 20px;
}

#forContenido select {
    width: 224px;
    padding: 5px 10px 5px 10px;
    background-color: white;
    border: 1px solid gray;
    color: black;
    margin-bottom: 20px;
}

#forContenido input[type="date"] {
    width: 202px;
    padding: 5px 10px 5px 10px;
    background-color: white;
    border: 1px solid gray;
    color: black;
    margin-bottom: 20px;
}

#forContenido button {
  background-color: #2ecc71; /* Verde */
    font-size: 14px;
    font-weight: 400;
    padding: 7px 10px;
    color: #fff;
    border: none;
    border-radius: 4px;
    cursor: pointer;
}

#editDate {
    width: 201px;
    padding: 5px 10px 5px 10px;
}

#loteria {
    width: 67rem;
    height: 40rem;
    margin-left: 40px;
    margin-right: 40px;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 10px;
    overflow-y: auto;
}

::-webkit-scrollbar {
    width: 6px;
}

::-webkit-scrollbar-thumb {
    background-color: #95a5a6;
    border-radius: 3px;
}

::-webkit-scrollbar-track {
    background-color: #ecf0f1;
}

#loteria button {
    border-radius: 50%;
    width: 1px;
    height: 1px;
    padding: 25px;
    display: flex;
    justify-content: center;
    align-items: center;
   
}

#boletaDesplegableReserva {
    display: flex;
    flex-direction: column;
    background-color: #fff;
    color: beige;
    width: 18rem;
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translate(-50%);
}

#boletaDesplegableReserva h3 {
    margin: 10px 0 5px 0;
}

#boletaDesplegableReserva p {
    margin: 5px 0 10px 0;
}

#boletaDesplegableReserva div {
    display: flex;
    flex-direction: column;
}

#boletaDesplegableReserva div button {
    background-color: #fff;
    color: #000;
    text-align: left;
}

#boletaDesplegableDisponible {
    display: flex;
    flex-direction: column;
    background-color: #fff;
    color: black;
    width: 18rem;
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translate(-50%);
}

#boletaDesplegableDisponible h3 {
    margin: 10px 0 5px 0;
}

#boletaDesplegableDisponible p {
    margin: 5px 0 10px 0;
}

#boletaDesplegableDisponible div {

    display: flex;
    flex-direction: column;
}

#boletaDesplegableDisponible div button {
    background-color: #fff;
    color: #000;
    text-align: left;
}
</style>