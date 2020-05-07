<template>
	<v-layout align-start>
		<v-flex>
			<v-data-table
				:headers="headers"
				:items="proveedores"
				:search="search"
				sort-by="calories"
				class="elevation-1"
			>
				<template v-slot:top>
					<v-toolbar text color="white">
						<v-toolbar-title>Proveedores</v-toolbar-title>
						<v-divider class="mx-4" inset vertical></v-divider>
						<v-spacer></v-spacer>
						<v-text-field
							class="text-xs-center"
							v-model="search"
							append-icon="search"
							label="Búsqueda"
							single-line
							hide-details
						></v-text-field>
						<v-spacer></v-spacer>
						<v-dialog v-model="dialog" max-width="500px">
							<template v-slot:activator="{ on }">
								<v-btn color="primary" dark class="mb-2" v-on="on">Nuevo</v-btn>
							</template>
							<v-card>
								<v-card-title>
									<span class="headline">{{ formTitle }}</span>
								</v-card-title>

								<v-card-text>
									<v-container>
										<v-row>
											<v-col cols="12" sm="12" md="12">
												<v-text-field v-model="nombre" label="Nombre"></v-text-field>
											</v-col>
											<v-col cols="6" sm="6" md="6">
												<v-select v-model="tipo_documento" :items="documentos " label="Tipo Documento"></v-select>
											</v-col>
											<v-col cols="6" sm="6" md="6">
												<v-text-field v-model="num_documento" label="Número Documento"></v-text-field>
											</v-col>
											<v-col cols="12" sm="12" md="12">
												<v-text-field  v-model="direccion" label="Dirección"></v-text-field>
											</v-col>
											<v-col cols="6" sm="6" md="6">
												<v-text-field  v-model="telefono" label="Teléfono"></v-text-field>
											</v-col>
											<v-col cols="6" sm="6" md="6">
												<v-text-field  v-model="email" label="Email"></v-text-field>
											</v-col>
											<v-col cols="12" sm="12" md="12" v-show="valida">
												<div
													class="red--text"
													v-for="v in validaMensaje"
													:key="v"
													v-text="v"
												></div>
											</v-col>
										</v-row>
									</v-container>
								</v-card-text>

								<v-card-actions>
									<v-spacer></v-spacer>
									<v-btn color="blue darken-1" text @click="close">Cancelar</v-btn>
									<v-btn color="blue darken-1" text @click="guardar">Guardar</v-btn>
								</v-card-actions>
							</v-card>
						</v-dialog>
					</v-toolbar>
				</template>

				<template v-slot:item.opciones="{ item }">
					<v-icon small class="mr-2" @click="editItem(item)">
						edit
					</v-icon>
				</template>
				<template v-slot:no-data>
					<v-btn color="primary" @click="listar">Resetear</v-btn>
				</template>
			</v-data-table>
		</v-flex>
	</v-layout>
</template>

<script>
import axios from 'axios';
export default {
	data() {
		return {
			proveedores: [],
			dialog: false,
			headers: [
				{ text: 'Nombre', value: 'nombre' },
				{ text: 'Tipo Persona', value: 'tipo_persona' },
				{ text: 'Tipo Documento', value: 'tipo_documento' },
				{ text: 'Número Documento', value: 'num_documento', sortable: false }, 
				{ text: 'Dirección', value: 'direccion', sortable: false },
				{ text: 'Teléfono', value: 'telefono', sortable: false },
				{ text: 'Email', value: 'email', sortable: false },
				{ text: 'Opciones', value: 'opciones', sortable: false }
			],
			search: '',
			editedIndex: -1,
			id: '',
			nombre:'',
			tipo_documento:'',
			documentos:['DNI','RUC','PASAPORTE','CEDULA'],
			num_documento: '',
			direccion:'',
			telefono:'',
			email: '',
			valida: 0,
			validaMensaje: [],
			adModal: 0,
			adAccion: 0,
			adNombre: '',
			adId: '',
		};
	},

	computed: {
		formTitle() {
			return this.editedIndex === -1 ? 'Nuevo proveedor' : 'Actualizar proveedor';
		},
	},

	watch: {
		dialog(val) {
			val || this.close();
		},
	},

	created() {
		console.log('inicio');
		this.listar();
	},

	methods: {
		listar() {
			let me = this;
			axios
				.get('api/Personas/ListarProveedores') 	 	 	
				.then(function(response) {
					console.log(response);
					me.proveedores = response.data;
				})
				.catch(function(error) {
					console.log('error');
				});
		},

		editItem(item) {
			this.id = item.idpersona;
			this.nombre = item.nombre;
			this.tipo_documento=item.tipo_documento;
			this.num_documento=item.num_documento;
			this.direccion = item.direccion;
			this.telefono = item.telefono;
			this.email = item.email;
			this.editedIndex = 1;
			this.dialog = true
		},

		close() {
			this.dialog = false;
			this.limpiar();
		},
		limpiar() {
			this.id = "";
			this.nombre="";
			this.tipo_documento = "";
			this.num_documento="";
			this.direccion="";
			this.telefono="";
			this.email="";
			this.editedIndex = -1;
		},
		guardar() {
			if (this.validar()) {
				return;
			}
			if (this.editedIndex > -1) {
				//Código para editar
				let me = this;
				axios
					.put('api/Personas/Actualizar', {
						'idpersona':me.id,
                        'tipo_persona':'Proveedor',
						'nombre': me.nombre,
						'tipo_documento': me.tipo_documento,
						'num_documento':me.num_documento,
						'direccion':me.direccion,
						'telefono':me.telefono,
						'email':me.email
					})
					.then(function(response) {
						me.close();
						me.listar();
						me.limpiar();
					})
					.catch(function(error) {
						console.log(error);
					});
			} else {
				//Código para guardar
				let me = this;
				axios
					.post('api/Personas/Crear', {
						'tipo_persona':'Proveedor',
						'nombre': me.nombre,
						'tipo_documento': me.tipo_documento,
						'num_documento':me.num_documento,
						'direccion':me.direccion,
						'telefono':me.telefono,
						'email':me.email
					})
					.then(function(response) {
						me.close();
						me.listar();
						me.limpiar();
					})
					.catch(function(error) {
						console.log(error);
					});

					
			}
		},
		validar() {
			this.valida = 0;
			this.validaMensaje = [];
			if (this.nombre.length < 3 || this.nombre.length > 100) {
				this.validaMensaje.push('El nombre deber tener más de 3 caracteres y menos de 100 caracteres.');
			}
			if(!this.tipo_documento){
				this.validaMensaje.push('Seleccione un tipo de documento.');
			}
			if (this.validaMensaje.length) {
				this.valida = 1;
			}
			return this.valida;
		},

	},
};
</script>
