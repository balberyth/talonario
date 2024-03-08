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
let mostrarParticipante = ref(false);
let mostrarListarBoletas = ref(false);
let mostrarFondo = ref(true);
let mostrarPerzonalicacion = ref(false);

let boletaReservada = ref(false);
let boletaDisponible = ref(false);
let boletaComprada = ref(false);

let boletas = ref([]);

// Personalizacion de colores
let colorFondo = ref("#ffffff");
let colorBoletasCompradas = ref("#0000FF");
let colorBoletasReservadas = ref("#FF0000");
let colorBoletasDisponibles = ref("#000000");
let colorPagina = ref("#0000FF");

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

		if (boletas.value.length == 0) {
			for (let i = 0; i < 100; i++) {
				boletas.value.push({
					i,
					estado: "Disponible",
				});
			}
		}
		mostrarFormulario.value = false;
		mostrarFondo.value = false;
	}
}

function editar() {
	mostrarFormulario.value = true;
	mostrarInformacion.value = false;
	mostrarFondo.value = true;
}

let element = ref(null);
let index = ref(null);

function modificarBoleta(elemento, indice) {
	if (elemento.estado == "Disponible") {
		// mostrar desplegable de boletas disponible
		boletaDisponible.value = false;
		boletaDisponible.value = true;
		// esconde demas desplegables
		boletaReservada.value = false;
		boletaComprada.value = false;

		element.value = elemento;
		index.value = indice;
	} else if (elemento.estado == "Reservado") {
		// mostrar desplegable de boletas reservada
		boletaReservada.value = false;
		boletaReservada.value = true;
		// esconde demas desplegables
		boletaDisponible.value = false;
		boletaComprada.value = false;

		element.value = elemento;
		index.value = indice;
	} else {
		// mostrar desplegable de boletas comprada
		boletaComprada.value = false;
		boletaComprada.value = true;
		// esconde demas desplegables
		boletaDisponible.value = false;
		boletaReservada.value = false;

		element.value = elemento;
		index.value = indice;
	}
}

function mostrarFBD() {
	mostrarFormularioBD.value = false;
	mostrarFormularioBD.value = true;
	mostrarFondo.value = true;
}

let clientes = ref([]);
let textDireccion = /^[A-Za-z0-9\s]+$/g;

let hoyAhoraParaAdquirir = null;

function adquirir() {
	hoyAhoraParaAdquirir = new Date();
	let fechaTexto = `${hoyAhoraParaAdquirir.getFullYear()}/${
		hoyAhoraParaAdquirir.getMonth() + 1
	}/${hoyAhoraParaAdquirir.getDate()}`;

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
	} else if (!isNaN(nombreBD.value)) {
		Swal.fire({
			text: "Ingrese un nombre valido",
			icon: "warning",
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
	} else if (!textDireccion.test(direccionBD.value)) {
		Swal.fire({
			text: "Ingrese una dirección de residencia valida",
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
			fecha: fechaTexto,
			indice: index.value,
		});
		boletas.value[index.value].estado = "Comprado";
		mostrarFormularioBD.value = false;
		mostrarFondo.value = false;
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
			fecha: fechaTexto,
			indice: index.value,
		});
		boletas.value[index.value].estado = "Reservado";
		mostrarFormularioBD.value = false;
		mostrarFondo.value = false;
		boletaDisponible.value = false;
	}
}

function cambiarColorBoton(objeto) {
	switch (objeto) {
		case "Disponible":
			return { backgroundColor: colorBoletasDisponibles.value };
		case "Reservado":
			return { backgroundColor: colorBoletasReservadas.value };
		case "Comprado":
			return { backgroundColor: colorBoletasCompradas.value };
	}
}

let participante = null;

function mostrarDatosParticipante() {
	mostrarParticipante.value = true;
	mostrarFondo.value = true;
	participante = clientes.value.find((c) => c.indice == index.value);
}

function mostrarPerzonalizar() {
    mostrarPerzonalicacion.value = true;
	mostrarFondo.value = true;
}

function liberarBoleta() {
	boletas.value[index.value].estado = "Disponible";
	modificarBoleta(element.value, index.value);
}

function marcarPagada() {
	boletas.value[index.value].estado = "Comprado";
	modificarBoleta(element.value, index.value);
}

function cerrarVentanaParticipante() {
	mostrarParticipante.value = false;
	mostrarFondo.value = false;
}

function cerrarVentanaPersonalizar() {
	mostrarPerzonalicacion.value = false;
	mostrarFondo.value = false;
}

function mostrarLista() {
	mostrarListarBoletas.value = true;
	mostrarFondo.value = true;
}

function cerrarLista() {
	mostrarListarBoletas.value = false;
	mostrarFondo.value = false;
}

const imprimir = () => {
	const doc = new jsPDF();

	doc.setFontSize(12);
	doc.text("Resumen de boletas vendidas", 10, 10);

	const tableData = clientes.value.map((boleta, index) => [
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

	const totalBoletas = clientes.value.length;
	doc.text(
		`Total de balotas compradas: ${totalBoletas}`,
		10,
		doc.autoTable.previous.finalY + 10
	);

	doc.save("vendidas.pdf");
};
</script>

<template>
	<div id="all" :style="{ backgroundColor: colorFondo }">
		<div id="header" :style="{ backgroundColor: colorPagina }">
			<h1>TALONARIO</h1>
		</div>
		<div id="footer" :style="{ backgroundColor: colorPagina }">
			<footer>© Copyright 2024 - Todos los derechos reservados</footer>
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
				<button @click="mostrarDatosParticipante()">
					Ver datos del participante
				</button>
				<button @click="liberarBoleta()">Liberar boleta</button>
				<button @click="marcarPagada()">Marcar como pagada</button>
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
				<button @click="mostrarDatosParticipante()">
					Ver datos del participante
				</button>
				<button @click="liberarBoleta()">Liberar boleta</button>
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
						🏆<span>{{
							mostrarInformacion == true ? premio : ""
						}}</span>
					</p>
					<p>
						💰<span>{{
							mostrarInformacion == true ? precioBoleta : ""
						}}</span>
					</p>
					<p>
						🏦<span>{{
							mostrarInformacion == true ? tipoLoteria : ""
						}}</span>
					</p>
					<p>
						🗓<span>{{
							mostrarInformacion == true ? fechaSorteo : ""
						}}</span>
					</p>
					<button
						@click="editar()"
						:style="{ backgroundColor: colorPagina }"
					>
						Editar ✏
					</button>
				</div>
			</section>
			<section id="loteria">
				<button
					v-for="(e, i) in boletas"
					:key="i"
					@click="modificarBoleta(e, i)"
					:style="cambiarColorBoton(e.estado)"
				>
					{{ e.i < 10 ? "0" + e.i : e.i }}
				</button>
			</section>
			<section id="accion">
				<h2>ACCIONES</h2>
				<div id="accContenido">
					<button id="estadoB">ESTADO</button>
					<button
						@click="mostrarLista()"
						:style="{ backgroundColor: colorPagina }"
					>
						LISTAR BOLETAS
					</button>
					<button @click="mostrarPerzonalizar()" :style="{ backgroundColor: colorPagina }">
						PERSONALIZAR TALONARIO WEB
					</button>
					<button
						@click="imprimir()"
						:style="{ backgroundColor: colorPagina }"
					>
						GENERAR ARCHIVO DE DATOS
					</button>
				</div>
			</section>
			<!-- Fondo gris -->
			<div id="fondo" v-if="mostrarFondo == true"></div>
			<!-- Formulario boleta disponible -->
			<section id="formularioBD" v-if="mostrarFormularioBD == true">
				<h2 :style="{ backgroundColor: colorPagina }">
					DILIGENCIA LA INFORMACION
				</h2>
				<p>
					Boleta a Adquirir:
					<span>{{ index <= 9 ? "0" + index : index }}</span>
				</p>
				<input
					type="text"
					id="dBD"
					placeholder="🧍🏻Nombre"
					v-model="nombreBD"
				/>
				<input
					type="tel"
					placeholder="📞 Telefono"
					v-model="telefonoBD"
				/>
				<input
					type="text"
					placeholder="🏠 Direccion"
					v-model="direccionBD"
				/>
				<p>Pagar ya:</p>
				<div>
					<label for="pagarSi">
						<input
							type="radio"
							name="pagar"
							id="pagarSi"
							value="si"
							v-model="pagarBD"
						/>
						Si
					</label>
					<label for="pagarSi">
						<input
							type="radio"
							name="pagar"
							id="pagarNo"
							value="no"
							v-model="pagarBD"
						/>
						No
					</label>
				</div>
				<button
					@click="adquirir()"
					:style="{ backgroundColor: colorPagina }"
				>
					ADQUIRIR
				</button>
			</section>
			<!-- Formulario registro talonario -->
			<section id="formulario" v-if="mostrarFormulario == true">
				<h2 :style="{ backgroundColor: colorPagina }">
					CONFIGURA TU TALONARIO
				</h2>
				<div id="forContenido">
					<input type="text" placeholder="Premio" v-model="premio" />
					<input
						type="text"
						placeholder="Valor Boleta"
						v-model="precioBoleta"
					/>
					<select v-model="tipoLoteria">
						<option disabled value="">
							Selecciona una lotería
						</option>
						<option value="Baloto" label="Baloto"></option>
						<option value="La Culona" label="La Culona"></option>
						<option
							value="Lotería de Bogotá"
							label="Lotería de Bogotá"
						></option>
						<option
							value="Lotería de Boyacá"
							label="Lotería de Boyacá"
						></option>
						<option
							value="Lotería del Huila"
							label="Lotería del Huila"
						></option>
						<option
							value="Lotería de Medellín"
							label="Lotería de Medellín"
						></option>
						<option
							value="Lotería del Tolima"
							label="Lotería del Tolima"
						></option>
						<option
							value="Lotería del Valle"
							label="Lotería del Valle"
						></option>
						<option
							value="Super Astro"
							label="Super Astro"
						></option>
					</select>
					<input type="date" id="editDate" v-model="fechaSorteo" />
					<button
						@click="generarLoteria()"
						:style="{ backgroundColor: colorPagina }"
					>
						GUARDAR
					</button>
				</div>
			</section>
		</main>
		<div id="participante" v-if="mostrarParticipante == true">
			<h2 :style="{ backgroundColor: colorPagina }">PARTICIPANTE</h2>
			<div id="contenedor">
				<section>
					<p class="icono">👤</p>
					<p class="texto">
						NOMBRE <span>{{ participante.nombre }}</span>
					</p>
				</section>
				<section>
					<p class="icono">📱</p>
					<p class="texto">
						TELEFONO <span>{{ participante.telefono }}</span>
					</p>
				</section>
				<section>
					<p class="icono">🏠</p>
					<p class="texto">
						DIRECCIÓN <span>{{ participante.direccion }}</span>
					</p>
				</section>
				<section>
					<p class="icono">📅</p>
					<p class="texto">
						FECHA <span>{{ participante.fecha }}</span>
					</p>
				</section>
				<section id="final">
					<p class="icono">💾</p>
					<p class="texto">
						ESTADO
						<span>{{
							participante.pagar == "si" ? "Pagado" : "Por pagar"
						}}</span>
					</p>
					<button id="cerrar" @click="cerrarVentanaParticipante()">❌</button>
				</section>
			</div>
		</div>
		<div id="listarBoletas" v-if="mostrarListarBoletas == true">
			<h2 :style="{ backgroundColor: colorPagina }">
				LISTADO DE BOLETAS
			</h2>
			<div>
				<article v-for="(e, i) in clientes" :key="i">
					<p>Nombre</p>
					<span>{{ e.nombre }}</span>
					<p>Teléfono</p>
					<span>{{ e.telefono }}</span>
					<p>Dirección</p>
					<span>{{ e.direccion }}</span>
					<p>Boletas</p>
					<span>{{ e.indice }}</span>
				</article>
			</div>
			<button
				@click="cerrarLista()"
				:style="{ backgroundColor: colorPagina }"
			>
				Cerrar
			</button>
		</div>
		<div id="paletasDeColores" v-if="mostrarPerzonalicacion == true">
			<h2 :style="{ backgroundColor: colorPagina }">
				PALETAS DE COLORES
			</h2>
			<section>
				<article>
					<!-- Uploaded to: SVG Repo, www.svgrepo.com, Generator: SVG Repo Mixer Tools -->
					<svg
						width="100px"
						height="100px"
						viewBox="0 0 24 24"
						fill="none"
						stroke="#000000"
						stroke-width="0.500"
						xmlns="http://www.w3.org/2000/svg"
					>
						<path
							d="M17.3108 11.25C17.3308 11.51 17.2408 11.78 17.0408 11.98L11.0208 18C9.69083 19.33 8.35083 19.33 7.01083 18L3.00083 13.99C2.32083 13.3 1.98083 12.61 2.00083 11.92H2.07083L17.1908 11.26L17.3108 11.25Z"
							:fill="colorFondo"
						/>
						<path
							opacity="0.4"
							d="M17.04 10.6402L9.69 3.29013L8.82 2.42014C8.53 2.13014 8.05 2.13014 7.76 2.42014C7.47 2.71014 7.47 3.19013 7.76 3.48013L8.63 4.35013L3 9.98013C2.36 10.6201 2.02 11.2701 2 11.9201H2.07L17.19 11.2602L17.31 11.2502C17.3 11.0302 17.2 10.8002 17.04 10.6402Z"
							fill="#292D32"
						/>
						<path
							d="M16 22.75H3C2.59 22.75 2.25 22.41 2.25 22C2.25 21.59 2.59 21.25 3 21.25H16C16.41 21.25 16.75 21.59 16.75 22C16.75 22.41 16.41 22.75 16 22.75Z"
							:fill="colorFondo"
						/>
						<path
							d="M19.35 14.7798C19.09 14.4998 18.61 14.4998 18.35 14.7798C18.04 15.1198 16.5 16.8598 16.5 18.1698C16.5 19.4698 17.55 20.5198 18.85 20.5198C20.15 20.5198 21.2 19.4698 21.2 18.1698C21.2 16.8598 19.66 15.1198 19.35 14.7798Z"
							:fill="colorFondo"
						/>
					</svg>
					<div>
						<p>Color fondo</p>
						<input
							type="color"
							v-model="colorFondo"
						/>
					</div>
				</article>
				<article>
					<svg
						width="100px"
						height="100px"
						viewBox="0 0 24 24"
						fill="none"
						stroke="#000000"
						stroke-width="0.500"
						xmlns="http://www.w3.org/2000/svg"
					>
						<path
							d="M17.3108 11.25C17.3308 11.51 17.2408 11.78 17.0408 11.98L11.0208 18C9.69083 19.33 8.35083 19.33 7.01083 18L3.00083 13.99C2.32083 13.3 1.98083 12.61 2.00083 11.92H2.07083L17.1908 11.26L17.3108 11.25Z"
							:fill="colorBoletasCompradas"
						/>
						<path
							opacity="0.4"
							d="M17.04 10.6402L9.69 3.29013L8.82 2.42014C8.53 2.13014 8.05 2.13014 7.76 2.42014C7.47 2.71014 7.47 3.19013 7.76 3.48013L8.63 4.35013L3 9.98013C2.36 10.6201 2.02 11.2701 2 11.9201H2.07L17.19 11.2602L17.31 11.2502C17.3 11.0302 17.2 10.8002 17.04 10.6402Z"
							fill="#292D32"
						/>
						<path
							d="M16 22.75H3C2.59 22.75 2.25 22.41 2.25 22C2.25 21.59 2.59 21.25 3 21.25H16C16.41 21.25 16.75 21.59 16.75 22C16.75 22.41 16.41 22.75 16 22.75Z"
							:fill="colorBoletasCompradas"
						/>
						<path
							d="M19.35 14.7798C19.09 14.4998 18.61 14.4998 18.35 14.7798C18.04 15.1198 16.5 16.8598 16.5 18.1698C16.5 19.4698 17.55 20.5198 18.85 20.5198C20.15 20.5198 21.2 19.4698 21.2 18.1698C21.2 16.8598 19.66 15.1198 19.35 14.7798Z"
							:fill="colorBoletasCompradas"
						/>
					</svg>
					<div>
						<p>Color Boletas Compradas</p>
						<input
							type="color"
							v-model="colorBoletasCompradas"
						/>
					</div>
				</article>
				<article>
					<svg
						width="100px"
						height="100px"
						viewBox="0 0 24 24"
						fill="none"
						stroke="#000000"
						stroke-width="0.500"
						xmlns="http://www.w3.org/2000/svg"
					>
						<path
							d="M17.3108 11.25C17.3308 11.51 17.2408 11.78 17.0408 11.98L11.0208 18C9.69083 19.33 8.35083 19.33 7.01083 18L3.00083 13.99C2.32083 13.3 1.98083 12.61 2.00083 11.92H2.07083L17.1908 11.26L17.3108 11.25Z"
							:fill="colorBoletasReservadas"
						/>
						<path
							opacity="0.4"
							d="M17.04 10.6402L9.69 3.29013L8.82 2.42014C8.53 2.13014 8.05 2.13014 7.76 2.42014C7.47 2.71014 7.47 3.19013 7.76 3.48013L8.63 4.35013L3 9.98013C2.36 10.6201 2.02 11.2701 2 11.9201H2.07L17.19 11.2602L17.31 11.2502C17.3 11.0302 17.2 10.8002 17.04 10.6402Z"
							fill="#292D32"
						/>
						<path
							d="M16 22.75H3C2.59 22.75 2.25 22.41 2.25 22C2.25 21.59 2.59 21.25 3 21.25H16C16.41 21.25 16.75 21.59 16.75 22C16.75 22.41 16.41 22.75 16 22.75Z"
							:fill="colorBoletasReservadas"
						/>
						<path
							d="M19.35 14.7798C19.09 14.4998 18.61 14.4998 18.35 14.7798C18.04 15.1198 16.5 16.8598 16.5 18.1698C16.5 19.4698 17.55 20.5198 18.85 20.5198C20.15 20.5198 21.2 19.4698 21.2 18.1698C21.2 16.8598 19.66 15.1198 19.35 14.7798Z"
							:fill="colorBoletasReservadas"
						/>
					</svg>
					<div>
						<p>Color Boletas Reservadas</p>
						<input
							type="color"
							v-model="colorBoletasReservadas"
						/>
					</div>
				</article>
				<article>
					<svg
						width="100px"
						height="100px"
						viewBox="0 0 24 24"
						fill="none"
						stroke="#000000"
						stroke-width="0.500"
						xmlns="http://www.w3.org/2000/svg"
					>
						<path
							d="M17.3108 11.25C17.3308 11.51 17.2408 11.78 17.0408 11.98L11.0208 18C9.69083 19.33 8.35083 19.33 7.01083 18L3.00083 13.99C2.32083 13.3 1.98083 12.61 2.00083 11.92H2.07083L17.1908 11.26L17.3108 11.25Z"
							:fill="colorBoletasDisponibles"
						/>
						<path
							opacity="0.4"
							d="M17.04 10.6402L9.69 3.29013L8.82 2.42014C8.53 2.13014 8.05 2.13014 7.76 2.42014C7.47 2.71014 7.47 3.19013 7.76 3.48013L8.63 4.35013L3 9.98013C2.36 10.6201 2.02 11.2701 2 11.9201H2.07L17.19 11.2602L17.31 11.2502C17.3 11.0302 17.2 10.8002 17.04 10.6402Z"
							fill="#292D32"
						/>
						<path
							d="M16 22.75H3C2.59 22.75 2.25 22.41 2.25 22C2.25 21.59 2.59 21.25 3 21.25H16C16.41 21.25 16.75 21.59 16.75 22C16.75 22.41 16.41 22.75 16 22.75Z"
							:fill="colorBoletasDisponibles"
						/>
						<path
							d="M19.35 14.7798C19.09 14.4998 18.61 14.4998 18.35 14.7798C18.04 15.1198 16.5 16.8598 16.5 18.1698C16.5 19.4698 17.55 20.5198 18.85 20.5198C20.15 20.5198 21.2 19.4698 21.2 18.1698C21.2 16.8598 19.66 15.1198 19.35 14.7798Z"
							:fill="colorBoletasDisponibles"
						/>
					</svg>
					<div>
						<p>Color Boletas Disponibles</p>
						<input
							type="color"
							v-model="colorBoletasDisponibles"
						/>
					</div>
				</article>
				<article id="final2">
					<svg
						width="100px"
						height="100px"
						viewBox="0 0 24 24"
						fill="none"
						stroke="#000000"
						stroke-width="0.500"
						xmlns="http://www.w3.org/2000/svg"
					>
						<path
							d="M17.3108 11.25C17.3308 11.51 17.2408 11.78 17.0408 11.98L11.0208 18C9.69083 19.33 8.35083 19.33 7.01083 18L3.00083 13.99C2.32083 13.3 1.98083 12.61 2.00083 11.92H2.07083L17.1908 11.26L17.3108 11.25Z"
							:fill="colorPagina"
						/>
						<path
							opacity="0.4"
							d="M17.04 10.6402L9.69 3.29013L8.82 2.42014C8.53 2.13014 8.05 2.13014 7.76 2.42014C7.47 2.71014 7.47 3.19013 7.76 3.48013L8.63 4.35013L3 9.98013C2.36 10.6201 2.02 11.2701 2 11.9201H2.07L17.19 11.2602L17.31 11.2502C17.3 11.0302 17.2 10.8002 17.04 10.6402Z"
							fill="#292D32"
						/>
						<path
							d="M16 22.75H3C2.59 22.75 2.25 22.41 2.25 22C2.25 21.59 2.59 21.25 3 21.25H16C16.41 21.25 16.75 21.59 16.75 22C16.75 22.41 16.41 22.75 16 22.75Z"
							:fill="colorPagina"
						/>
						<path
							d="M19.35 14.7798C19.09 14.4998 18.61 14.4998 18.35 14.7798C18.04 15.1198 16.5 16.8598 16.5 18.1698C16.5 19.4698 17.55 20.5198 18.85 20.5198C20.15 20.5198 21.2 19.4698 21.2 18.1698C21.2 16.8598 19.66 15.1198 19.35 14.7798Z"
							:fill="colorPagina"
						/>
					</svg>
					<div>
						<p>Color De La Pagina</p>
						<input
							type="color"
							v-model="colorPagina"
						/>
					</div>
				</article>
			</section>
            <button id="cerrar2" @click="cerrarVentanaPersonalizar()">❌</button>
		</div>
	</div>
</template>

<style scoped>
#all {
	width: 100rem;
	position: absolute;
	left: 0;
	top: 0;
	width: 100%;
	height: 100vh;
}

#header {
	position: fixed;
	top: 0;
	left: 50%;
	transform: translate(-50%);
	font-size: 7px;
	padding: 5px 0 5px 0;
	width: 100%;
	box-shadow: 0 5px 10px rgba(0, 0, 0, 0.26);
}

#footer {
	position: fixed;
	bottom: 0;
	left: 50%;
	transform: translate(-50%);
	padding: 5px 0 5px 0;
	width: 100%;
}

main {
	padding: 130px 0 0 0;
	display: flex;
	flex-direction: row;
	justify-content: center;
}

#informacion {
	margin-top: 10rem;
	color: rgb(0, 0, 0);
	background-color: rgba(113, 187, 247, 0.795);
	border-radius: 20px;
	padding: 0 15px 15px 15px;
	width: 250px;
	height: 290px;
}

#informacion h2 {
	font-weight: 400;
	font-size: 15px;
}

#infoContenido {
	text-align: start;
	background-color: white;
	box-shadow: rgba(0, 0, 0, 0.16) 0px 3px 6px, rgba(0, 0, 0, 0.23) 0px 3px 6px;
	padding: 5px 15px 15px 15px;
	border-radius: 5px;
}

#informacion button {
	background-color: rgb(108, 223, 127) !important;
	position: relative;
	left: 50%;
	transform: translate(-50%);
	font-size: 14px;
	font-weight: 400;
	padding: 5px 10px 7px 10px;
	
	
}

#accion {
	margin-top: 10rem;
	color: #000000;
	background-color:  rgba(113, 187, 247, 0.795);
	border-radius: 20px;
	padding: 0 15px 15px 15px;
	width: 250px;
	height: 290px;
}

#accion h2 {
	font-weight: 400;
	font-size: 15px;
}

#accContenido {
	text-align: center;
	background-color: white;
	box-shadow: rgba(0, 0, 0, 0.16) 0px 3px 6px, rgba(0, 0, 0, 0.23) 0px 3px 6px;
	padding: 5px 15px 15px 15px;
	border-radius: 5px;
	font-size: 12px;
}

#accion button {
	width: 140px;
	margin-bottom: 5px;
	border-radius: 6px;

}

#accion #estadoB {
	margin-top: 21px;
	margin-bottom: 10px;
	background-color: rgb(108, 223, 127);
	border-radius: 2px;
}

#fondo {
	position: absolute;
	background-color: #034be600;
	width: 100%;
	height: 10vh;
	top: 0;
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
	font-size: 13px;
	width: 100px;
	padding: 5px 10px 5px 10px;
	border-radius: 0;
	
}

#formulario {
	display: flex;
	flex-direction: column;
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	align-items: center;
	background-color: aliceblue;
	border: 1px solid black;
	border-radius: 5px;
	box-shadow: rgba(0, 0, 0, 0.16) 0px 3px 6px, rgba(0, 0, 0, 0.23) 0px 3px 6px;
	
}

#formulario h2 {
	padding: 5px 40px;
	font-size: 15px;
	position: relative;
	bottom: 12.2px;
}

#forContenido {
	display: flex;
	flex-direction: column;
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
	font-size: 13px;
	width: 100px;
	padding: 5px 10px 5px 10px;
	border-radius: 0;
	position: relative;
	left: 50%;
	transform: translate(-50%);
	margin-bottom: 20px;
	background-color: rgb(108, 223, 127) !important;
}

#editDate {
	width: 201px;
	padding: 5px 10px 5px 10px;
}

#loteria {
	width: 40rem;
	height: 30rem;
	margin-left: 40px;
	margin-right: 40px;
	display: flex;
	flex-wrap: wrap;
	justify-content: space-between;
	gap: 10px;
	overflow-y: auto;
	background-color: rgba(77, 206, 223, 0.041);
}

::-webkit-scrollbar {
	width: 7px;
}

::-webkit-scrollbar-thumb {
	background-color: #2251e9;
	border-radius: 3px;
}

::-webkit-scrollbar-track {
	background-color: #a8e0f000;
}

#loteria button {
	border-radius: 50%;
	width: 1px;
	height: 1px;
	padding: 25px;
	display: flex;
	justify-content: center;
	align-items: center;
	background-color: beige !important;
	color:#000;
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

#participante {
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	width: 35rem;
	display: flex;
	flex-direction: column;
	background-color: #fff;
}

#participante h2 {
	color: #fff;
	padding: 5px 40px;
	font-size: 15px;
	position: relative;
	bottom: 12.2px;
}

#contenedor {
	display: grid;
	grid-template-columns: 1fr 1fr;
	color: #000;
}

#contenedor p {
	font-weight: 700;
}

#contenedor span {
	font-weight: 400;
}

#final {
	grid-column: 1/3;
}

#contenedor section {
	display: flex;
	flex-direction: row;
	justify-content: center;
}

.texto {
	display: flex;
	flex-direction: column;
}

.icono {
	line-height: 30px;
	margin-right: 10px;
	font-size: 25px;
}

#cerrar {
	background-color: transparent;
	font-size: 16px;
	position: absolute;
	top: 0;
	right: 0;
	padding: 3px 5px 0 0;
}

#cerrar2 {
	background-color: transparent;
	font-size: 16px;
	position: absolute;
	top: 0;
	right: 0;
	padding: 3px 5px 0 0;
}

#listarBoletas {
	background-color: #fff;
	color: #000;
	position: absolute;
	top: 50%;
	left: 50%;
	transform: translate(-50%, -50%);
	width: 82rem;
}

#listarBoletas div {
	display: flex;
	flex-direction: row;
	flex-wrap: wrap;
	gap: 10px;
	justify-content: start;
	background-color: #fff;
	height: 20rem;
	overflow-y: auto;
	padding: 10px 6px 10px 10px;
	border-bottom: 3px solid #ccc;
}

#listarBoletas h2 {
	color: #fff;
	padding: 10px 40px;
	font-size: 15px;
	text-align: left;
	position: relative;
	bottom: 12.2px;
}

#listarBoletas article {
	display: flex;
	flex-direction: column;
	align-items: start;
	box-shadow: rgba(14, 30, 37, 0.12) 0px 2px 4px 0px,
		rgba(14, 30, 37, 0.32) 0px 2px 16px 0px;
	width: 250px;
	padding-top: 8px;
}

#listarBoletas article p {
	display: flex;
	flex-direction: column;
	margin: 0;
	margin-left: 10px;
}

#listarBoletas article span {
	font-weight: 600;
	margin-bottom: 10px;
	margin-left: 10px;
}

#listarBoletas button {
	margin: 10px;
	background-color:rgb(108, 223, 127) !important ;
}

#paletasDeColores {
	background-color: #fff;
	color: #000;
	position: absolute;
	top: 100%;
	left: 50%;
	transform: translate(-50%, -100%);
	width: 45rem;
	height: 32rem;
}

#paletasDeColores h2 {
	color: #fff;
	padding: 10px 40px;
	font-size: 15px;
	text-align: left;
	position: relative;
	bottom: 12.2px;
}

#paletasDeColores section {
	display: flex;
	flex-direction: row;
	flex-wrap: wrap;
	justify-content: center;
	align-content: center;
	gap: 50px;
	color: #000;
	
}

#paletasDeColores section article {
	display: flex;
	flex-direction: row;
	width: 200px;
	gap: 20px;
}

#paletasDeColores section article div {
	display: flex;
	flex-direction: column;
	justify-content: center;
}
</style>