<script setup>
import { ref } from "vue";
import Swal from "sweetalert2";
import jsPDF from "jspdf";
import autoTable from "jspdf-autotable";

jsPDF.autotable = autoTable;

// Formulario
let premio = ref("");
let precioBoleta = ref("");
let tipoLoteria = ref("");
let fechaSorteo = ref("");

// FormularioCliente
let nombreCliente = ref("");
let telefonoCliente = ref("");
let direccionCliente = ref("");
const pagarCliente = ref('si');

// Fechas
let hoy = new Date();
hoy.setHours(0, 0, 0, 0);
let fechaDate = ref(null)

// Mostrar 
let mostrarFormulario = ref(true);
let mostrarBoletas = ref(false);
let mostrarRegistroDatosDeBoleta = ref(false);
let datosParticipanteVisible = ref(false);
let mostrarListadoBoletas = ref(false);
let mostrarFondo = ref(true);
let mostrarPerzonalicacion = ref(false);

// Estados de Boleta
let boletaReservada = ref(false);
let boletaDisponible = ref(false);
let boletaComprada = ref(false);

// Arrays
let boletas = ref([]);
let clientes = ref([]);

// Item e Índice del array Boletas de las boletas
let item = ref(null);
let index = ref(null);

// Item del cliente
const participante = ref(null);

// Personalizacion de colores
let colorFondo = ref("#ffffff");
let colorBoletasCompradas = ref("#0000FF");
let colorBoletasReservadas = ref("#FF0000");
let colorBoletasDisponibles = ref("#000000");
let colorPagina = ref("#0000FF");

// Funciones Lógicas
function generarLoteria() {
    fechaDate = new Date(fechaSorteo.value + "T00:00:00");

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
    } else if (fechaDate <= hoy) {
        Swal.fire({
            text: "Ingrese una fecha final válida",
            icon: "warning",
        });
    } else {
        mostrarBoletas.value = true;
        Swal.fire({
            text: "¡Gracias por usar este Talonario!",
            icon: "success",
        });
        if (boletas.value.length == 0) {
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


function editarBalota(item, indice) {
    // Ocultar todos los desplegables
    boletaReservada.value = false;
    boletaDisponible.value = false;
    boletaComprada.value = false;
    datosParticipanteVisible.value = false;
    mostrarRegistroDatosDeBoleta.value = false;
    
    // Mostrar el desplegable correspondiente al estado de la boleta
    switch (item.estado) {
        case "Disponible":
            boletaDisponible.value = true;
            break;
            case "Reservado":
                boletaReservada.value = true;
            break;
        case "Comprado":
            boletaComprada.value = true;
            break;
        }
        item = item;
        index = indice;
    }
    
    function mostrarformulario() {
        mostrarRegistroDatosDeBoleta.value = true;
        boletaDisponible.value = false;
    }
    
    function editar() {
        mostrarFormulario.value = true;
        mostrarBoletas.value = false;
    }
function adquirir() {
    var soloLetras = /^[A-Za-z]+$/;
    
    if (
        nombreCliente.value == "" ||
        telefonoCliente.value == "" ||
        direccionCliente.value == "" ||
        pagarCliente.value == ""
        ) {
            Swal.fire({
                text: "Los datos están vacíos",
                icon: "error",
        });
    } else if (!soloLetras.test(nombreCliente.value)) {
        Swal.fire({
            text: "El nombre debe contener solo letras",
            icon: "error",
        });
    } else if (isNaN(telefonoCliente.value)) {
        Swal.fire({
            text: "Solo números en telefono",
            icon: "warning",
        });
    } else {
        // Ensure that pagarCliente is always set to "si" before processing
        pagarCliente.value = "si";

        // Search for an existing client
        let clienteExistente = clientes.value.find(
            (cliente) =>
                cliente.nombre === nombreCliente.value &&
                cliente.telefono === telefonoCliente.value &&
                cliente.direccion === direccionCliente.value
        );

        if (pagarCliente.value == "si") {
            // Process purchase
            if (clienteExistente) {
                Swal.fire({
                    text: "¡Gracias por comprar otra boleta!",
                    icon: "success",
                });
                clienteExistente.boletas.push(index);
            } else {
                Swal.fire({
                    text: "¡Gracias por comprar la boleta!",
                    icon: "success",
                });
                clientes.value.push({
                    nombre: nombreCliente.value,
                    telefono: telefonoCliente.value,
                    direccion: direccionCliente.value,
                    pagar: pagarCliente.value,
                    boletas: [index],
                    indice: index,
                });
            }
            boletas.value[index].estado = 'Comprado';
        } else if (pagarCliente.value == "no") {
            // Process reservation
            if (clienteExistente) {
                Swal.fire({
                    text: "¡Gracias por reservar otra boleta!",
                    icon: "success",
                });
                clienteExistente.boletas.push(index);
            } else {
                Swal.fire({
                    text: "¡Gracias por reservar una boleta!",
                    icon: "success",
                });
                clientes.value.push({
                    nombre: nombreCliente.value,
                    telefono: telefonoCliente.value,
                    direccion: direccionCliente.value,
                    pagar: pagarCliente.value,
                    boletas: [index],
                    indice: index,
                });
            }
            boletas.value[index].estado = 'Reservado';
        }

        // Reset form fields
        nombreCliente.value = "";
        telefonoCliente.value = "";
        direccionCliente.value = "";
        mostrarRegistroDatosDeBoleta.value = false;
        boletaDisponible.value = false;
    }
}

function cambiarColorBoton(objeto) {
    switch (objeto) {
        case "Disponible":
            return { backgroundColor: "beige", color: "black" };
        case "Reservado":
            return { backgroundColor: "red", color: "white" };
        case "Comprado":
            return { backgroundColor: "blue" };
    }
}

const verDatosParticipante = () => {
    participante.value = clientes.value.find(cliente => cliente.boletas.includes(index));
    datosParticipanteVisible.value = true;
    boletaComprada.value = false;
    boletaReservada.value = false;
    document.getitemById('volverBtn').style.display = 'block';
};

const BotonCerrar = () => {
    mostrarRegistroDatosDeBoleta.value = false;
    boletaReservada.value = false;
    boletaComprada.value = false;
    boletaDisponible.value = false;
    mostrarPerzonalicacion.value = false;
    mostrarFondo.value = false;
};

function volver() {
    datosParticipanteVisible.value = false;
    boletaComprada.value = false;
    boletaReservada.value = false;

    if (item.estado === 'Reservado') {
        boletaReservada.value = true;

    } else if (item.estado === 'Comprado') {
        boletaComprada.value = true;
    }
}

function liberar() {
    let estadoActualizado = 'Disponible';

    if (item.estado === "Reservado" || item.estado === "Comprado") {
        boletaReservada.value = false;
        boletaComprada.value = false;

        // Obtener el índice de la boleta en la lista de clientes
        let clienteIndex = clientes.value.findIndex(
            (cliente) => cliente.boletas.includes(index)
        );

        // Eliminar la boleta de la lista del cliente
        if (clienteIndex !== -1) {
            clientes.value[clienteIndex].boletas = clientes.value[clienteIndex].boletas.filter(
                (boletaIndex) => boletaIndex !== index
            );
        }
        participante.value = null;
    } else {
        estadoActualizado = 'Disponible';
    }
    Swal.fire({
        text: "¡Boleta liberada exitosamente!",
        icon: "success",
    });
    boletas.value[index].estado = estadoActualizado;
}

function marcarComoPagada() {
    boletaReservada.value = false;
    boletas.value[index].estado = 'Comprado';
    Swal.fire({
        text: "¡Boleta marcada como pagada exitosamente!",
        icon: "success"
    });
}

function mostrarListadoDeBoletas() {
    mostrarListadoBoletas.value = true;
}
function cerrarListadoBoletas() {
    mostrarListadoBoletas.value = false;
}
function mostrarPerzonalizar() {
    mostrarPerzonalicacion.value = true;
    mostrarFondo.value = true;
}

const imprimir = () => {
  const doc = new jsPDF();

  doc.setFontSize(12);
  doc.text("Resumen de boletas vendidas", 10, 10);

  // Filter out deleted boletas
  const filteredBoletas = clientes.value.filter((boleta) => !boleta.deleted);

  const tableData = filteredBoletas.map((boleta, index) => [
    boleta.indice,
    boleta.nombre,
    boleta.telefono,
    boleta.direccion,
    boleta.pagar,
  ]);

  doc.autoTable({
    head: [["Boleta", "Nombre", "Teléfono", "Dirección", "Pago"]],
    body: tableData,
    startY: 20,
  });

  const totalBoletas = filteredBoletas.length; // Update this line
  doc.text("Total de boletas compradas: ${totalBoletas}, 10, doc.autoTable.previous.finalY + 10");

  doc.save("vendidas.pdf");
};


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
            <button @click="BotonCerrar" class="cerrarBtn">x</button>
            <h3>
                Boleta
                <span>{{ item.i > 10 ? item.i : "0" + item.i }}</span>
            </h3>
            <p>
                Estado:
                <span>Reservada 🔴</span>
            </p>
            <div>
                <button @click="verDatosParticipante">Ver datos del participante</button>
                <button @click="liberar">Liberar boleta</button>
                <button @click="marcarComoPagada">Marcar como pagada</button>
            </div>
        </div>
        <!-- Div de datos del participante -->
        <div id="datosParticipante" v-if="datosParticipanteVisible">
            <button @click="volver()" class="cerrarBtnVolver">⮌</button>
            <h3>
                Boleta
                <span>{{ participante.indice > 10 ? participante.indice : '0' + participante.indice }}</span>
            </h3>
            <p>
                Estado:
                <span>{{ participante.pagar === 'si' ? 'Comprada 🔵' : 'Reservada 🔴' }}</span>
            </p>
            <div>
                <p><strong>Nombre:</strong> {{ participante.nombre }}</p>
                <p><strong>Teléfono:</strong> {{ participante.telefono }}</p>
                <p><strong>Dirección:</strong> {{ participante.direccion }}</p>
            </div>
        </div>
        <!-- Desplegable boleta comprada -->
        <div id="boletaDesplegableComprada" v-if="boletaComprada == true">
            <button @click="BotonCerrar" class="cerrarBtn">x</button>
            <h3>
                Boleta
                <span>{{ item.i > 10 ? item.i : "0" + item.i }}</span>
            </h3>
            <p>
                Estado:
                <span>Comprada 🔵</span>
            </p>
            <div>
                <button @click="verDatosParticipante">Ver datos del participante</button>
                <button @click="liberar">Liberar boleta</button>
            </div>
        </div>
        <!-- Desplegable boleta disponible -->
        <div id="boletaDesplegableDisponible" v-if="boletaDisponible == true">
            <button @click="BotonCerrar" class="cerrarBtn">x</button>
            <h3>
                Boleta
                <span>{{ item.i > 10 ? item.i : "0" + item.i }}</span>
            </h3>
            <p>Estado: <span>Disponible ⚫</span></p>
            <div>
                <button @click="mostrarformulario()">🤝 Adquirir Boleta</button>
            </div>
        </div>
        <main>
            <section id="informacion">
                <h2 style="color: white">INFORMACIÓN</h2>
                <div id="infoContenido">
                    <p>
                        🏆<span>{{ mostrarBoletas == true ? premio : "" }}</span>
                    </p>
                    <p>
                        💰<span>{{ mostrarBoletas == true ? precioBoleta : "" }}</span>
                    </p>
                    <p>
                        🏦<span>{{ mostrarBoletas == true ? tipoLoteria : "" }}</span>
                    </p>
                    <p>
                        🗓️<span>{{ mostrarBoletas == true ? fechaSorteo : "" }}</span>
                    </p>
                    <button @click="editar()">Editar ✏️</button>
                </div>
            </section>
            <section id="loteria">
                <button v-for="(e, i) in  boletas " :key="i" @click="editarBalota(e, i)"
                    :style="cambiarColorBoton(e.estado)">
                    {{ e.i < 10 ? "0" + e.i : e.i }} </button>
            </section>
            <section id="accion">
                <h2 style="color: white">ACCIONES</h2>
                <div id="accContenido">
                    <button id="estadoB">ESTADO</button>
                    <button @click="mostrarListadoDeBoletas">LISTAR BOLETAS</button>
                    <button @click="mostrarPerzonalizar()" :style="{ backgroundColor: colorPagina }">PERSONALIZAR
                        TALONARIO WEB</button>
                    <button @click="imprimir()" :style="{ backgroundColor: colorPagina }">GENERAR ARCHIVO DE
                        DATOS</button>
                </div>
            </section>

            <section v-if="mostrarListadoBoletas" class="listado-boletas">
                <h2>LISTADO DE BOLETAS</h2>
                <div class="papaCard">
                    <div class="card"
                        v-for="(cliente, index) in clientes.filter(cliente => cliente.boletas.some(boletaIndex => boletas[boletaIndex].estado === 'Comprado'))"
                        :key="index">
                        <p>Nombre</p>
                        <p class="textoListar">{{ cliente.nombre }}</p>
                        <p>Telefono</p>
                        <p class="textoListar">{{ cliente.telefono }}</p>
                        <p>Direccion</p>
                        <p class="textoListar">{{ cliente.direccion }}</p>
                        <p>Boletas</p>
                        <p> {{ cliente.boletas.filter(boletaIndex => boletas[boletaIndex].estado === 'Comprado')
            .join(' - ') }} </p>
                    </div>
                </div>
                <div class="lin"></div>
                <button class="cerrarr" @click="cerrarListadoBoletas">Cerrar</button>
            </section>

            <div id="fondo" v-if="mostrarFormulario == true"></div>
            <section id="formularioBD" v-if="mostrarRegistroDatosDeBoleta == true">
                <button @click="BotonCerrar" class="cerrar">x</button>
                <h2>DILIGENCIA LA INFORMACION</h2>
                <p>
                    Boleta a Adquirir:
                    <span>{{ index <= 9 ? "0" + index : index }}</span>
                </p>
                <input type="text" id="dBD" placeholder="🧍🏻Nombre" v-model="nombreCliente" />
                <input type="tel" placeholder="📞 Telefono" v-model="telefonoCliente" />
                <input type="text" placeholder=" Direccion" v-model="direccionCliente" />
                <p>Pagar ya:</p>
                <div>
                    <label for="pagarSi">
                        <input type="radio" name="pagar" id="pagarSi" value="si" v-model="pagarCliente" />
                        Si
                    </label>
                    <label for="pagarNo">
                        <input type="radio" name="pagar" id="pagarNo" value="no" v-model="pagarCliente" />
                        No
                    </label>
                </div>
                <button class="adquirir" @click="adquirir()">ADQUIRIR</button>
            </section>
            <section id="formulario" v-if="mostrarFormulario == true">
                <h2 style="color: white">CONFIGURA TU TALONARIO</h2>
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
            <div id="paletasDeColores" v-if="mostrarPerzonalicacion == true">
                <h2 :style="{ backgroundColor: colorPagina, margin: 0 }">
                    PALETAS DE COLORES
                </h2>
                <section>
                    <article>
                        <!-- Uploaded to: SVG Repo, www.svgrepo.com, Generator: SVG Repo Mixer Tools -->
                        <svg width="100px" height="100px" viewBox="0 0 24 24" fill="none" stroke="#000000"
                            stroke-width="0.500" xmlns="http://www.w3.org/2000/svg">
                            <path
                                d="M17.3108 11.25C17.3308 11.51 17.2408 11.78 17.0408 11.98L11.0208 18C9.69083 19.33 8.35083 19.33 7.01083 18L3.00083 13.99C2.32083 13.3 1.98083 12.61 2.00083 11.92H2.07083L17.1908 11.26L17.3108 11.25Z"
                                :fill="colorFondo" />
                            <path opacity="0.4"
                                d="M17.04 10.6402L9.69 3.29013L8.82 2.42014C8.53 2.13014 8.05 2.13014 7.76 2.42014C7.47 2.71014 7.47 3.19013 7.76 3.48013L8.63 4.35013L3 9.98013C2.36 10.6201 2.02 11.2701 2 11.9201H2.07L17.19 11.2602L17.31 11.2502C17.3 11.0302 17.2 10.8002 17.04 10.6402Z"
                                fill="#292D32" />
                            <path
                                d="M16 22.75H3C2.59 22.75 2.25 22.41 2.25 22C2.25 21.59 2.59 21.25 3 21.25H16C16.41 21.25 16.75 21.59 16.75 22C16.75 22.41 16.41 22.75 16 22.75Z"
                                :fill="colorFondo" />
                            <path
                                d="M19.35 14.7798C19.09 14.4998 18.61 14.4998 18.35 14.7798C18.04 15.1198 16.5 16.8598 16.5 18.1698C16.5 19.4698 17.55 20.5198 18.85 20.5198C20.15 20.5198 21.2 19.4698 21.2 18.1698C21.2 16.8598 19.66 15.1198 19.35 14.7798Z"
                                :fill="colorFondo" />
                        </svg>
                        <div>
                            <p>Color fondo</p>
                            <input type="color" v-model="colorFondo" />
                        </div>
                    </article>
                    <article>
                        <svg width="100px" height="100px" viewBox="0 0 24 24" fill="none" stroke="#000000"
                            stroke-width="0.500" xmlns="http://www.w3.org/2000/svg">
                            <path
                                d="M17.3108 11.25C17.3308 11.51 17.2408 11.78 17.0408 11.98L11.0208 18C9.69083 19.33 8.35083 19.33 7.01083 18L3.00083 13.99C2.32083 13.3 1.98083 12.61 2.00083 11.92H2.07083L17.1908 11.26L17.3108 11.25Z"
                                :fill="colorBoletasCompradas" />
                            <path opacity="0.4"
                                d="M17.04 10.6402L9.69 3.29013L8.82 2.42014C8.53 2.13014 8.05 2.13014 7.76 2.42014C7.47 2.71014 7.47 3.19013 7.76 3.48013L8.63 4.35013L3 9.98013C2.36 10.6201 2.02 11.2701 2 11.9201H2.07L17.19 11.2602L17.31 11.2502C17.3 11.0302 17.2 10.8002 17.04 10.6402Z"
                                fill="#292D32" />
                            <path
                                d="M16 22.75H3C2.59 22.75 2.25 22.41 2.25 22C2.25 21.59 2.59 21.25 3 21.25H16C16.41 21.25 16.75 21.59 16.75 22C16.75 22.41 16.41 22.75 16 22.75Z"
                                :fill="colorBoletasCompradas" />
                            <path
                                d="M19.35 14.7798C19.09 14.4998 18.61 14.4998 18.35 14.7798C18.04 15.1198 16.5 16.8598 16.5 18.1698C16.5 19.4698 17.55 20.5198 18.85 20.5198C20.15 20.5198 21.2 19.4698 21.2 18.1698C21.2 16.8598 19.66 15.1198 19.35 14.7798Z"
                                :fill="colorBoletasCompradas" />
                        </svg>
                        <div>
                            <p>Color Boletas Compradas</p>
                            <input type="color" v-model="colorBoletasCompradas" />
                        </div>
                    </article>
                    <article>
                        <svg width="100px" height="100px" viewBox="0 0 24 24" fill="none" stroke="#000000"
                            stroke-width="0.500" xmlns="http://www.w3.org/2000/svg">
                            <path
                                d="M17.3108 11.25C17.3308 11.51 17.2408 11.78 17.0408 11.98L11.0208 18C9.69083 19.33 8.35083 19.33 7.01083 18L3.00083 13.99C2.32083 13.3 1.98083 12.61 2.00083 11.92H2.07083L17.1908 11.26L17.3108 11.25Z"
                                :fill="colorBoletasReservadas" />
                            <path opacity="0.4"
                                d="M17.04 10.6402L9.69 3.29013L8.82 2.42014C8.53 2.13014 8.05 2.13014 7.76 2.42014C7.47 2.71014 7.47 3.19013 7.76 3.48013L8.63 4.35013L3 9.98013C2.36 10.6201 2.02 11.2701 2 11.9201H2.07L17.19 11.2602L17.31 11.2502C17.3 11.0302 17.2 10.8002 17.04 10.6402Z"
                                fill="#292D32" />
                            <path
                                d="M16 22.75H3C2.59 22.75 2.25 22.41 2.25 22C2.25 21.59 2.59 21.25 3 21.25H16C16.41 21.25 16.75 21.59 16.75 22C16.75 22.41 16.41 22.75 16 22.75Z"
                                :fill="colorBoletasReservadas" />
                            <path
                                d="M19.35 14.7798C19.09 14.4998 18.61 14.4998 18.35 14.7798C18.04 15.1198 16.5 16.8598 16.5 18.1698C16.5 19.4698 17.55 20.5198 18.85 20.5198C20.15 20.5198 21.2 19.4698 21.2 18.1698C21.2 16.8598 19.66 15.1198 19.35 14.7798Z"
                                :fill="colorBoletasReservadas" />
                        </svg>
                        <div>
                            <p>Color Boletas Reservadas</p>
                            <input type="color" v-model="colorBoletasReservadas" />
                        </div>
                    </article>
                    <article>
                        <svg width="100px" height="100px" viewBox="0 0 24 24" fill="none" stroke="#000000"
                            stroke-width="0.500" xmlns="http://www.w3.org/2000/svg">
                            <path
                                d="M17.3108 11.25C17.3308 11.51 17.2408 11.78 17.0408 11.98L11.0208 18C9.69083 19.33 8.35083 19.33 7.01083 18L3.00083 13.99C2.32083 13.3 1.98083 12.61 2.00083 11.92H2.07083L17.1908 11.26L17.3108 11.25Z"
                                :fill="colorBoletasDisponibles" />
                            <path opacity="0.4"
                                d="M17.04 10.6402L9.69 3.29013L8.82 2.42014C8.53 2.13014 8.05 2.13014 7.76 2.42014C7.47 2.71014 7.47 3.19013 7.76 3.48013L8.63 4.35013L3 9.98013C2.36 10.6201 2.02 11.2701 2 11.9201H2.07L17.19 11.2602L17.31 11.2502C17.3 11.0302 17.2 10.8002 17.04 10.6402Z"
                                fill="#292D32" />
                            <path
                                d="M16 22.75H3C2.59 22.75 2.25 22.41 2.25 22C2.25 21.59 2.59 21.25 3 21.25H16C16.41 21.25 16.75 21.59 16.75 22C16.75 22.41 16.41 22.75 16 22.75Z"
                                :fill="colorBoletasDisponibles" />
                            <path
                                d="M19.35 14.7798C19.09 14.4998 18.61 14.4998 18.35 14.7798C18.04 15.1198 16.5 16.8598 16.5 18.1698C16.5 19.4698 17.55 20.5198 18.85 20.5198C20.15 20.5198 21.2 19.4698 21.2 18.1698C21.2 16.8598 19.66 15.1198 19.35 14.7798Z"
                                :fill="colorBoletasDisponibles" />
                        </svg>
                        <div>
                            <p>Color Boletas Disponibles</p>
                            <input type="color" v-model="colorBoletasDisponibles" />
                        </div>
                    </article>
                    <article id="final2">
                        <svg width="100px" height="100px" viewBox="0 0 24 24" fill="none" stroke="#000000"
                            stroke-width="0.500" xmlns="http://www.w3.org/2000/svg">
                            <path
                                d="M17.3108 11.25C17.3308 11.51 17.2408 11.78 17.0408 11.98L11.0208 18C9.69083 19.33 8.35083 19.33 7.01083 18L3.00083 13.99C2.32083 13.3 1.98083 12.61 2.00083 11.92H2.07083L17.1908 11.26L17.3108 11.25Z"
                                :fill="colorPagina" />
                            <path opacity="0.4"
                                d="M17.04 10.6402L9.69 3.29013L8.82 2.42014C8.53 2.13014 8.05 2.13014 7.76 2.42014C7.47 2.71014 7.47 3.19013 7.76 3.48013L8.63 4.35013L3 9.98013C2.36 10.6201 2.02 11.2701 2 11.9201H2.07L17.19 11.2602L17.31 11.2502C17.3 11.0302 17.2 10.8002 17.04 10.6402Z"
                                fill="#292D32" />
                            <path
                                d="M16 22.75H3C2.59 22.75 2.25 22.41 2.25 22C2.25 21.59 2.59 21.25 3 21.25H16C16.41 21.25 16.75 21.59 16.75 22C16.75 22.41 16.41 22.75 16 22.75Z"
                                :fill="colorPagina" />
                            <path
                                d="M19.35 14.7798C19.09 14.4998 18.61 14.4998 18.35 14.7798C18.04 15.1198 16.5 16.8598 16.5 18.1698C16.5 19.4698 17.55 20.5198 18.85 20.5198C20.15 20.5198 21.2 19.4698 21.2 18.1698C21.2 16.8598 19.66 15.1198 19.35 14.7798Z"
                                :fill="colorPagina" />
                        </svg>
                        <div>
                            <p>Color De La Pagina</p>
                            <input type="color" v-model="colorPagina" />
                        </div>
                    </article>
                </section>
                <button class="cerrarUltimoBtn" @click="BotonCerrar">x</button>
            </div>
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
    background-color: #3498db;
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
    color: #333;
    /* Cambia este color al que desees */
    padding-left: 3px;
}

#informacion button {
    background-color: #2ecc71;
    /* Verde */
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
    background-color: #2ecc71;
    /* Verde */
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
    margin: 0;
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

.adquirir {
    background-color: blue;
    font-size: 13px;
    width: 100px;
    padding: 5px 10px 5px 10px;
    border-radius: 0;
}

#formulario {
    margin-top: 10rem;
    color: black;
    background-color: #3498db;
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
    background-color: #2ecc71;
    /* Verde */
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
    color: black;
    width: 18rem;
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translate(-50%);
    padding-top: 20px;
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

#datosParticipante {
    display: flex;
    flex-direction: column;
    background-color: #fff;
    color: black;
    width: 18rem;
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translate(-50%);
    padding-top: 20px;
}

#datosParticipante .cerrarBtnVolver {
    position: absolute;
    top: -5px;
    left: 0;
    cursor: pointer;
    background: none;
    border: none;
    font-size: 1.5rem;
    color: red;
}


#datosParticipante h3 {
    margin: 10px 0 5px 0;
}

#datosParticipante p {
    margin: 5px 0 10px 0;
}

#datosParticipante p strong {
    font-weight: bold;
}

.cerrarBtn {
    position: absolute;
    top: -20px;
    right: -15px;
    cursor: pointer;
    background: none;
    border: none;
    font-size: 2rem;
    color: red;
    width: 2px;
}

.cerrar {
    margin: 0;
    position: absolute;
    top: -30.5px;
    right: -27px;
    cursor: pointer;
    background: none;
    border: none;
    font-size: 2rem;
    color: red;
    z-index: 999;
    background-color: transparent;
    /* Agrega !important para asegurar que se aplique */
}


#boletaDesplegableComprada {
    display: flex;
    flex-direction: column;
    background-color: #fff;
    color: black;
    width: 18rem;
    position: absolute;
    bottom: 0;
    left: 50%;
    transform: translate(-50%);
    padding-top: 20px;
}

#boletaDesplegableComprada h3 {
    margin: 10px 0 5px 0;
}

#boletaDesplegableComprada p {
    margin: 5px 0 10px 0;
}

#boletaDesplegableComprada div {
    display: flex;
    flex-direction: column;
}

#boletaDesplegableComprada div button {
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
    position: fixed;
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

.listado-boletas {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: white;
    box-shadow: rgba(17, 17, 26, 0.1) 0px 4px 16px, rgba(17, 17, 26, 0.05) 0px 8px 32px;
    z-index: 999;
    color: black;
    min-width: 30rem;
    max-width: 61rem;
    height: max-content;
}

.listado-boletas h2 {
    margin: 0;
    width: 98%;
    height: 60px;
    background-color: blue;
    color: white;
    text-align: left;
    line-height: 60px;
    padding-left: 20px;
}

.listado-boletas p {
    margin: 0;
}

.papaCard {
    display: flex;
    flex-direction: row;
    align-items: center;
    justify-content: center;
    flex-wrap: wrap;
}

.card {
    box-shadow: rgba(0, 0, 0, 0.4) 0px 2px 4px, rgba(0, 0, 0, 0.3) 0px 7px 13px -3px, rgba(0, 0, 0, 0.2) 0px -3px 0px inset;
    margin: 10px;
    width: 240px;
    text-align: left;
    height: max-content;
}

.textoListar {
    font-weight: 600;
}

.lin {
    width: 100%;
    background-color: #33333363;
    height: 3px;
    margin-bottom: 5px;
    margin-top: 30px;
}

.cerrarr {
    margin-bottom: 5px;
}

#paletasDeColores {
    position: fixed;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    background-color: white;
    box-shadow: rgba(17, 17, 26, 0.1) 0px 4px 16px, rgba(17, 17, 26, 0.05) 0px 8px 32px;
    z-index: 999;
    color: black;
    width: 61rem;
    height: max-content;
    /* Cambié 'max-content' por '70vh' para que tenga un 70% del alto de la ventana */
}

#paletasDeColores section {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    padding: 20px;
    border-bottom: 1px solid #ddd;
    /* Línea divisoria entre artículos */
}

#paletasDeColores article {
    width: calc(25% - 20px);
    /* Ajusta el ancho de cada artículo según tu diseño */
}

.cerrarUltimoBtn {
    position: absolute;
    top: -38px;
    right: -38px;
    cursor: pointer;
    background: none;
    border: none;
    font-size: 2.5rem;
    color: red;
}
</style>
