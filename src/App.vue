<script setup>
import { ref } from "vue";
import Swal from "sweetalert2";
import jsPDF from "jspdf";
import autoTable from "jspdf-autotable";

jsPDF.autotable = autoTable;

// Personalizacion de colores
let colorFondo = ref("#9CDDDC");
let ColorBdisponible = ref("#F5F5DC");
let colorPagina = ref("#3498db");

// Arrays
let boletas = ref([]);
let clientes = ref([]);

// Fechas
let fechaDate = ref(null)
let hoy = new Date();
hoy.setHours(0, 0, 0, 0);

// Mostrar 
let mostrarFormulario = ref(true);
let mostrarBoletas = ref(false);
let mostrarRegistroDatosDeBoleta = ref(false);
let datosParticipanteVisible = ref(false);
let mostrarListadoBoletas = ref(false);
let mostrarFondo = ref(true);
let mostrarPerzonalicacion = ref(false);

// Formulario
let premio = ref("");
let precioBoleta = ref("");
let tipoLoteria = ref("");
let fechaSorteo = ref("");
let actualizarB=ref(false)

// Estados de Boleta
let boletaReservada = ref(false);
let boletaDisponible = ref(false);
let boletaComprada = ref(false);
let boletaGanada = ref(false);

// Item e Índice del array Boletas de las boletas
let element = ref(null);
let index = ref(null);

// FormularioCliente
let nombreCliente = ref("");
let telefonoCliente = ref("");
let direccionCliente = ref("");
let pagarCliente = ref("");

// Item del cliente
let p = ref(null);

// Lógica de Boleta Ganadora
const mostrarGanador = ref(false);
const boletaGanadora = ref("")
const estadoGanaste = ref(false);
const botonDeshabilitado = ref(false);

function formatearNumero(numero) {
    return numero.toString().replace(/\B(?=(\d{3})+(?!\d))/g, ".");
}

// Funciones Lógicas
function editarparticipantes() {
    mostrarFormulario.value = true;
    mostrarBoletas.value = false;
}
function formular() {
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
    } else if (fechaDate < hoy) {
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
function adquirir() {
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
    } else if (isNaN(telefonoCliente.value)) {
        Swal.fire({
            text: "Solo números en telefono",
            icon: "warning",
        });
    } else if (pagarCliente.value == "no") {
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
        boletas.value[index].estado = 'Reservado';

        nombreCliente.value = "";
        telefonoCliente.value = "";
        direccionCliente.value = "";
        pagarCliente.value = "";
        mostrarRegistroDatosDeBoleta.value = false;
        boletaDisponible.value = false;

    } else if (pagarCliente.value == "si") {
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
        boletas.value[index].estado = 'Comprado';

        nombreCliente.value = "";
        telefonoCliente.value = "";
        direccionCliente.value = "";
        pagarCliente.value = "";
        mostrarRegistroDatosDeBoleta.value = false;
        boletaDisponible.value = false;
    }
}
function color(objeto) {
    switch (objeto) {
        case "Disponible":
            return { backgroundColor: ColorBdisponible.value, color: "black" };
        case "Reservado":
            return { backgroundColor: "red", color: "white" };
        case "Comprado":
            return { backgroundColor: "blue", color: "white" };
        case 'Ganador':
            return { backgroundColor: 'green', color: 'white' };
    }
}
const verDatosParticipante = () => {
    p.value = clientes.value.find(cliente => cliente.boletas.includes(index));
    datosParticipanteVisible.value = true;
    boletaComprada.value = false;
    boletaReservada.value = false;
};
function formulario() {
    mostrarRegistroDatosDeBoleta.value = true;
    boletaDisponible.value = false;
}
const BotonCerrar = () => {
    switch (true) {
        case mostrarPerzonalicacion.value === true:
            mostrarPerzonalicacion.value = false;
            break;
        case mostrarRegistroDatosDeBoleta.value === true:
            mostrarRegistroDatosDeBoleta.value = false;
            break;
        case boletaComprada.value === true:
            boletaComprada.value = false;
            break;
        case boletaReservada.value === true:
            boletaReservada.value = false;
            break;
        case boletaDisponible.value === true:
            boletaDisponible.value = false;
            break;
        case boletaGanada.value === true:
            boletaGanada.value = false;
        case mostrarGanador.value === true:
            mostrarGanador.value = false;
            break;
        case estadoGanaste.value === true:
            estadoGanaste.value = false;
            break;
    }
};
function volver() {
    datosParticipanteVisible.value = false;
    boletaComprada.value = false;
    boletaReservada.value = false;

    if (element.estado === 'Reservado') {
        boletaReservada.value = true;

    } else if (element.estado === 'Comprado') {
        boletaComprada.value = true;
    }
}
function liberar() {
    // Verificar si la boleta está actualmente reservada o comprada
    if (boletas.value[index].estado === "Reservado" || boletas.value[index].estado === "Comprado") {
        boletaReservada.value = false;
        boletaComprada.value = false;

        // Encontrar el índice del cliente que tiene esta boleta
        let clienteIndex = clientes.value.findIndex(
            (cliente) => cliente.boletas.includes(index)
        );
        // Eliminar la boleta de la lista de boletas del cliente
        if (clienteIndex !== -1) {
            clientes.value[clienteIndex].boletas = clientes.value[clienteIndex].boletas.filter(
                (boletaIndex) => boletaIndex !== index
            );
            // Verificar si el cliente ya no tiene más boletas, luego eliminar al cliente
            if (clientes.value[clienteIndex].boletas.length === 0) {
                clientes.value.splice(clienteIndex, 1);
            }
        }

        // Restablecer el valor del participante
        p.value = null;
    }
    // Actualizar el estado de la boleta a 'Disponible'
    boletas.value[index].estado = 'Disponible';

    Swal.fire({
        text: "¡Boleta liberada exitosamente!",
        icon: "success",
    });
}
function marcarPagado() {
    boletaReservada.value = false;
    boletas.value[index].estado = 'Comprado';

    // Encuentra al participante al que pertenece la boleta
    const participanteIndex = clientes.value.findIndex
        (cliente => cliente.boletas.includes(index));
    // Actualiza el estado de la boleta en los datos del participante

    if (participanteIndex !== -1) {
        clientes.value[participanteIndex].pagar = 'si'; // Actualiza el estado del participante a "Comprada"
    }
    Swal.fire({
        text: "¡Boleta marcada como pagada exitosamente!",
        icon: "success"
    });
}
const pdf = () => {
    const doc = new jsPDF();
    doc.setFontSize(12);
    doc.text("Resumen de boletas vendidas", 10, 10);
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

    const totalBoletas = filteredBoletas.length;
    const totalPrecioBoletas = filteredBoletas.filter(boleta => boleta.pagar === 'si').length * precioBoleta.value;
    const totalPrecioFaltanteBoletasReservadas = filteredBoletas.filter(boleta => boleta.pagar === 'no').length * precioBoleta.value;

    doc.text(`Total de boletas compradas: ${totalBoletas}`, 10, doc.autoTable.previous.finalY + 10);
    doc.text(`Total del precio de las boletas compradas: ${formatearNumero(totalPrecioBoletas)}`, 10, doc.autoTable.previous.finalY + 20);
    doc.text(`Total del precio faltante de las boletas reservadas: ${formatearNumero(totalPrecioFaltanteBoletasReservadas)}`, 10, doc.autoTable.previous.finalY + 30);
    doc.save("vendidas.pdf");
};

function editarparticipante() {
	actualizarB.value = true;
	mostrarRegistroDatosDeBoleta.value = true;
	mostrarFondo.value = true;
	p = clientes.value.find((c) => c.indice == index);
    console.log(p, index.value);
	nombreCliente.value = p.nombre;
	telefonoCliente.value = p.telefono;
	direccionCliente.value = p.direccion;
	pagarCliente.value = p.pagar;
}

function actualizarpartipante (){
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
    } else if (isNaN(telefonoCliente.value)) {
        Swal.fire({
            text: "Solo números en telefono",
            icon: "warning",
        });
    } else if (pagarCliente.value == "no") {
        Swal.fire({
			text: "¡Gracias por reservar una boleta!",
			icon: "success",
		});

		clientes.value.find((cliente) => {
			if (cliente.indice === index) {
				cliente.nombre = nombreCliente.value;
				cliente.telefono = telefonoCliente.value;
				cliente.direccion = direccionCliente.value;
				cliente.pagar = pagarCliente.value;
			}
		});

		boletas.value[index].estado = "Reservado";
		actualizarB.value = false;
		mostrarRegistroDatosDeBoleta.value = false;
		mostrarFondo.value = false;
		boletaDisponible.value = false;

		editarBalota(element.value, index);

		nombreCliente = ref("");
		telefonoCliente = ref("");
		direccionCliente= ref("");
		pagarCliente = ref("");
	

    } else if (pagarCliente.value == "si") {
        Swal.fire({
			text: "¡La boleta ah sido actualizada!",
			icon: "success",
		});

		clientes.value.find((cliente) => {
			if (cliente.indice === index) {
				cliente.nombre = nombreCliente.value;
				cliente.telefono = telefonoCliente.value;
				cliente.direccion = direccionCliente.value;
				cliente.pagar = pagarCliente.value;
			}
		});
		
		boletas.value[index].estado = "Comprado";
		actualizarB.value = false;
		mostrarRegistroDatosDeBoleta.value = false;
		mostrarFondo.value = false;
		boletaDisponible.value = false;

		editarBalota(element.value, index);

		nombreCliente.value = "";
        telefonoCliente.value = "";
        direccionCliente.value = "";
        pagarCliente.value = "";
    
	}
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
function editarBalota(elemento, indice) {
    // Ocultar todos los desplegables
    boletaReservada.value = false;
    boletaDisponible.value = false;
    boletaComprada.value = false;
    boletaGanada.value = false;
    datosParticipanteVisible.value = false;
    mostrarRegistroDatosDeBoleta.value = false;
    nombreCliente.value = "";
    telefonoCliente.value = "";
    direccionCliente.value = "";
    pagarCliente.value = "";

    // Mostrar el desplegable correspondiente al estado de la boleta
    switch (elemento.estado) {
        case "Disponible":
            boletaDisponible.value = true;
            break;
        case "Reservado":
            boletaReservada.value = true;
            break;
        case "Comprado":
            boletaComprada.value = true;
            break;
        case "Ganador":
            boletaGanada.value = true
    }
    element = elemento;
    index = indice;
}
const agruparBoletasCompradas = () => {
    const clientesAgrupados = {};

    // Agrupar boletas por nombre, teléfono y dirección
    clientes.value.forEach((cliente) => {
        const clave = "${cliente.nombre}_${cliente.telefono}_${cliente.direccion}";
        if (!clientesAgrupados[clave]) {
            clientesAgrupados[clave] = { ...cliente, boletas: [cliente.boletas[0]] };
        } else {
            clientesAgrupados[clave].boletas.push(cliente.boletas[0]);
        }
    });

    return Object.values(clientesAgrupados);
};
function estadoGanador() {
    mostrarGanador.value = true;
}
function botonGanador() {
    mostrarGanador.value = false;
    estadoGanaste.value = true;
    botonDeshabilitado.value = true; // Deshabilita el botón después de hacer clic

    // Convierte boletaGanadora a número antes de comparar
    const boletaGanadoraNumero = parseInt(boletaGanadora.value);

    // Itera sobre todas las boletas
    boletas.value.forEach((e, index) => {
        // Comprueba si el número de boleta actual coincide con boletaGanadora
        if (e.i === boletaGanadoraNumero) {
            // Establece el estado de la boleta como "Ganador"
            boletas.value[index].estado = 'Ganador';
        }
    });
}

</script>

<template>
    <div v-if="mostrarGanador" class="divGanador">
        <button @click="BotonCerrar" class="cerrarBtn">x</button>
        <h3>¿Cuál será la boleta ganadora?</h3>
        La boleta ganadora es:
        <input type="number" v-model="boletaGanadora">
        <button class="buttonGanador" @click="botonGanador">Aceptar</button>
    </div>
    <div v-if="estadoGanaste" class="divGanaste">
        <button @click="BotonCerrar" class="cerrarBtn">x</button>
        <h3>Felicidades</h3>
        <h3>La boleta {{ boletaGanadora }} es la ganadora del premio de:</h3>
        <h3>Premio: &nbsp;{{ formatearNumero(premio) }}</h3>
    </div>
    <div id="all" :style="{ backgroundColor: colorFondo }">
        <div id="header" :style="{ backgroundColor: colorPagina }">
            <h1>TALONARIO</h1>
        </div>
        <div id="footer" :style="{ backgroundColor: colorPagina }">
            <footer>©️ Copyright 2024 - Todos los derechos reservados</footer>
        </div>
        <!-- Desplegable boleta reservada -->
        <div id="boletaDesplegableReserva" v-if="boletaReservada == true">
            <button @click="BotonCerrar" class="cerrarBtn">x</button>
            <h3>
                Boleta
                <span>{{ element.i > 10 ? element.i : "0" + element.i }}</span>
            </h3>
            <p>
                Estado:
                <span>Reservada 🔴</span>
            </p>
            <div>
                <button @click="verDatosParticipante">Ver datos del participante</button>
                <button @click="liberar">Liberar boleta</button>
                <button @click="marcarPagado">Marcar como pagada</button>
            </div>
        </div>
        <!-- Div de datos del participante -->
        <div id="datosParticipante" v-if="datosParticipanteVisible">
            <button @click="volver()" class="cerrarBtnVolver">⮌</button>
            <h3>
                Boleta
                <span>{{ p.indice > 10 ? p.indice : '0' + p.indice }}</span>
            </h3>
            <p>
                Estado:
                <span>{{ p.pagar === 'si' ? 'Comprada 🔵' : 'Reservada 🔴' }}</span>
            </p>
            <div>
                <p><strong>Nombre:</strong> {{ p.nombre }}</p>
                <p><strong>Teléfono:</strong> {{ p.telefono }}</p>
                <p><strong>Dirección:</strong> {{ p.direccion }}</p>
            </div>

            <button @click="editarparticipante()">editar datos </button>

            


        </div>
        <!-- Desplegable boleta comprada -->
        <div id="boletaDesplegableComprada" v-if="boletaComprada == true" :style="{ backgroundColor: boletaComprada }">
            <button @click="BotonCerrar" class="cerrarBtn">x</button>
            <h3>
                Boleta
                <span>{{ element.i > 10 ? element.i : "0" + element.i }}</span>
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
                <span>{{ element.i > 10 ? element.i : "0" + element.i }}</span>
            </h3>
            <p>Estado: <span>Disponible ⚫</span></p>
            <div>
                <button @click="formulario()">🤝 Adquirir Boleta</button>
            </div>
        </div>
        <!-- Desplegable boleta Ganada -->
        <div id="boletaDesplegableGanada" v-if="boletaGanada == true">
            <button @click="BotonCerrar" class="cerrarBtn">x</button>
            <h3>
                Boleta
                <span>{{ element.i > 10 ? element.i : "0" + element.i }}</span>
            </h3>
            <p>¡ FELICIDADES ERES EL GANADOR !</p>
            <p>Estado: <span>Ganador 🟢</span></p>
        </div>
        <main>
            <section id="informacion">
                <h2 style="color: white">INFORMACIÓN</h2>
                <div id="infoContenido">
                    <p>
                        🏆<span>{{ mostrarBoletas == true ? formatearNumero(premio) : '' }}</span>
                    </p>
                    <p>
                        💰<span>{{ mostrarBoletas == true ? formatearNumero(precioBoleta) : '' }}</span>
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
                <button v-for="(e, i) in boletas" :key="i" @click="editarBalota(e, i)" :style="color(e.estado)">
                    {{ e.i < 10 ? "0" + e.i : e.i }} </button>
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
                    <button @click="formular()">GUARDAR</button>
                </div>
            </section>
            <section id="accion">
                <h2 style="color: white">ACCIONES</h2>
                <div id="accContenido">
                    <button :disabled="botonDeshabilitado" @click="estadoGanador" id="estadoB">ESTADO GANADOR</button>
                    <button @click="mostrarListadoDeBoletas()">LISTAR BOLETAS</button>
                    <button @click="mostrarPerzonalizar()">PERSONALIZAR
                        TALONARIO WEB</button>
                    <button @click="pdf()">GENERAR ARCHIVO DE
                        DATOS</button>
                </div>
            </section>
            <section v-if="mostrarListadoBoletas" class="listado-boletas">
                <h2>&nbsp;&nbsp;&nbsp;&nbsp;LISTADO DE BOLETAS</h2>
                <div class="papaCard">
                    <div class="card" v-for="(cliente, index) in agruparBoletasCompradas()" :key="index">
                        <p>Nombre</p>
                        <p class="textoListar">{{ cliente.nombre }}</p>
                        <p>Telefono</p>
                        <p class="textoListar">{{ cliente.telefono }}</p>
                        <p>Direccion</p>
                        <p class="textoListar">{{ cliente.direccion }}</p>
                        <p>Boletas</p>
                        <p>{{ cliente.boletas.join(' - ') }}</p>
                    </div>
                </div>
                <div class="lin"></div>
                <button class="cerrarr" @click="cerrarListadoBoletas">Cerrar</button>
            </section>
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
                <button class="adquirir" @click="adquirir()" v-if="actualizarB==false">ADQUIRIR</button>
                <button class="adquirir" @click="actualizarpartipante()" v-else>actualizar</button>
            </section>

            <div id="paletasDeColores" v-if="mostrarPerzonalicacion == true">
                <h2 :style="{ margin: 0 }">
                    &nbsp;&nbsp;PALETAS DE COLORES<button @click="BotonCerrar">x</button>
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
                                :fill="ColorBdisponible" />
                            <path opacity="0.4"
                                d="M17.04 10.6402L9.69 3.29013L8.82 2.42014C8.53 2.13014 8.05 2.13014 7.76 2.42014C7.47 2.71014 7.47 3.19013 7.76 3.48013L8.63 4.35013L3 9.98013C2.36 10.6201 2.02 11.2701 2 11.9201H2.07L17.19 11.2602L17.31 11.2502C17.3 11.0302 17.2 10.8002 17.04 10.6402Z"
                                fill="#292D32" />
                            <path
                                d="M16 22.75H3C2.59 22.75 2.25 22.41 2.25 22C2.25 21.59 2.59 21.25 3 21.25H16C16.41 21.25 16.75 21.59 16.75 22C16.75 22.41 16.41 22.75 16 22.75Z"
                                :fill="ColorBdisponible" />
                            <path
                                d="M19.35 14.7798C19.09 14.4998 18.61 14.4998 18.35 14.7798C18.04 15.1198 16.5 16.8598 16.5 18.1698C16.5 19.4698 17.55 20.5198 18.85 20.5198C20.15 20.5198 21.2 19.4698 21.2 18.1698C21.2 16.8598 19.66 15.1198 19.35 14.7798Z"
                                :fill="ColorBdisponible" />
                        </svg>
                        <div>
                            <p>Color Boletas Disponibles</p>
                            <input type="color" v-model="ColorBdisponible" />
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
            </div>
        </main>
    </div>
</template>

<style scoped>
.divGanaste {
    z-index: 99;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    display: flex;
    flex-direction: column;
    flex-wrap: wrap;
    gap: 10px;
    padding-top: 10px;
    padding-bottom: 10px;
    padding-inline: 20px;
    background-color: white;
    color: black;
    align-items: center;
    border: 1px solid red;
    font-weight: 600;
    font-family: Cambria, Cochin, Georgia, Times, 'Times New Roman', serif;
}

.divGanaste h3 {
    margin: 0;
}

.buttonGanador {
    background-color: red;
    color: white;
}

.divGanador {
    z-index: 99;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    display: flex;
    flex-direction: column;
    flex-wrap: wrap;
    padding-inline: 40px;
    background-color: white;
    color: black;
    align-items: center;
    border: 1px solid red;
    padding-bottom: 15px;
}

.divGanador input {
    background-color: #333;
    width: 25px;
    color: white;
    padding-left: 5px;
    text-align: center;
    margin-bottom: 10px
}

.divGanador p {
    color: red;
    font-weight: 600;
}

#all {
    width: 100%;
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    height: 100vh;
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
    width: 100%;
    padding-bottom: 10px;
    box-shadow: 10px 10px 10px rgba(0, 0, 0, 0.26);
}

main {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    margin: auto;
    max-width: max-content;
    gap: 6%;
    padding: 0 7%;
    height: 100vh;
}

#informacion {
    color: black;
    background-color: #3498db;
    border-radius: 20px;
    max-width: 220px;
    max-height: 360px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    min-width: 230px;
    padding-bottom: 30px;
    margin-top: 2.3%;
}

#infoContenido {
    padding: 20px;
    text-align: start;
    background-color: beige;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    width: 50%;
    /* Ancho del contenido */
    max-height: 100%;
    /* Altura máxima relativa al contenedor padre */
    overflow-y: auto;
    font-size: 12px;
    /* Agregar barra de desplazamiento vertical si el contenido es demasiado largo */
}

#infoContenido p {
    display: flex;
    flex-direction: row;
}

#infoContenido span {
    color: #333;
    /* Cambia este color al que desees */
    padding-left: 3px;
    padding-right: 20px;
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
    color: black;
    background-color: #3498db;
    border-radius: 20px;
    max-height: 360px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    min-width: 230px;
    padding-bottom: 30px;
    margin-top: 2.3%;
}

#accion h2 {
    font-weight: 400;
    font-size: 20px;
}

#accContenido {
    padding: 20px;
    text-align: start;
    background-color: beige;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    max-height: 100%;
    overflow-y: auto;
}

#accion button {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    width: 140px;
    margin-bottom: 5px;
    border-radius: 6px;
    background-color: #2ecc71;
    color: #fff;
    border: none;
    cursor: pointer;
    font-size: 11px;
    max-height: 360px;
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
    color: black;
    background-color: #3498db;
    border-radius: 20px;
    padding: 20px;
    width: 240px;
    height: 350px;
    position: absolute;
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
    display: flex;
    flex-direction: column;
    align-items: center;
    justify-content: center;
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
    min-width: min-content;
    max-width: 120vh;
    display: flex;
    flex-wrap: wrap;
    justify-content: space-between;
    gap: 10px;
    overflow-y: auto;
    max-height: 60vh;
    margin-top: 2.3%;
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
    padding: 23px;
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
    z-index: 999;
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
    z-index: 999;
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
    z-index: 999;
    overflow-x: auto;
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
    z-index: 999;
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

#boletaDesplegableGanada {
    display: flex;
    flex-direction: column;
    background-color: #fff;
    color: black;
    width: 18rem;
    position: fixed;
    bottom: 0;
    left: 50%;
    transform: translate(-50%);
    z-index: 999;
}

#boletaDesplegableGanada h3 {
    margin: 9px 0 15px 0;
}

#boletaDesplegableGanada p {
    margin: 0px 0 9px 0;
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
    max-height: 30rem;
    overflow-y: auto;
}

.listado-boletas h2 {
    margin: 0;
    width: 100%;
    height: 60px;
    background-color: blue;
    color: white;
    text-align: left;
    line-height: 60px;
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
    font-weight: 700;
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
    background-color: rgba(255, 0, 0, 0.846);
    color: white;
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
    width: 41rem;
    width: 21rem;
    height: 19.6rem;
    display: flex;
    flex-direction: column;
}

#paletasDeColores h2 {
    text-align: left;
    background-color: rgba(0, 0, 255, 0.719);
}

#paletasDeColores section {
    flex: 1;
    /* Haz que la sección ocupe todo el espacio disponible en la dirección del eje principal */
    overflow-y: auto;
    display: flex;
    flex-direction: column;
}

#paletasDeColores article {
    height: auto;
    box-sizing: border-box;
    padding: 10px;
}

#paletasDeColores button {
    position: absolute;
    top: -31px;
    right: -31px;
    cursor: pointer;
    background: none;
    border: none;
    font-size: 2.2rem;
    color: red;
}
</style>
