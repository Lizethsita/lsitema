<template>
	<v-layout align-start>
		<v-flex>
			<v-data-table
				:headers="headers"
				:items="categorias"
				:search="search"
				sort-by="calories"
				class="elevation-1"
			>
				<template v-slot:top>
					<v-toolbar text color="white">
						<v-toolbar-title>Categorías</v-toolbar-title>
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
											<v-col cols="12" sm="12" md="12">
												<v-text-field v-model="descripcion" label="Descripción"></v-text-field>
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
						<v-dialog v-model="adModal" max-width="290">
							<v-card>
								<v-card-title class="headline" v-if="adAccion == 1">¿Activar Item?</v-card-title>
								<v-card-title class="headline" v-if="adAccion == 2">¿Desactivar Item?</v-card-title>
								<v-card-text>
									Estas a punto de
									<span v-if="adAccion == 1">Activar</span>
									<span v-if="adAccion == 2">Desactivar</span>
									el ítem {{ adNombre }}
								</v-card-text>
								<v-card-actions>
									<v-spacer></v-spacer>
									<v-btn color="green darken-1" text @click="activarDesactivarCerrar">Cancelar</v-btn>
									<v-btn v-if="adAccion==1" color="orange darken-4" text @click="activar">Activar</v-btn>
									<v-btn v-if="adAccion==2" color="orange darken-4" text @click="desactivar">Desactivar</v-btn>
								</v-card-actions>
							</v-card>
						</v-dialog>
					</v-toolbar>
				</template>

				<template v-slot:item.condicion="{ item }">
					<v-chip :color="getColor(item.condicion)" dark>
						<template v-if="item.condicion">
							Activo
						</template>
						<template v-else>
							Inactivo
						</template>
					</v-chip>
				</template>

				<template v-slot:item.opciones="{ item }">
					<v-icon small class="mr-2" @click="editItem(item)">
						edit
					</v-icon>
					<template v-if="item.condicion">
						<v-icon small @click="activarDesactivarMostrar(2, item)">
							block
						</v-icon>
					</template>
					<template v-else>
						<v-icon small @click="activarDesactivarMostrar(1, item)">
							check
						</v-icon>
					</template>
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
			categorias: [],
			dialog: false,
			headers: [
				{ text: 'Nombre', value: 'nombre' },
				{ text: 'Descripción', value: 'descripcion', sortable: false },
				{ text: 'Estado', value: 'condicion', sortable: false },
				{ text: 'Opciones', value: 'opciones', sortable: false },
			],
			search: '',
			editedIndex: -1,
			id: '',
			nombre: '',
			descripcion: '',
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
			return this.editedIndex === -1 ? 'Nueva categoría' : 'Actualizar categoría';
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
				.get('api/Categorias/Listar')
				.then(function(response) {
					console.log(response);
					me.categorias = response.data;
				})
				.catch(function(error) {
					console.log('error');
				});
		},
		editItem(item) {
			this.id = item.idcategoria;
			this.nombre = item.nombre;
			this.descripcion = item.descripcion;
			this.editedIndex = 1;
			this.dialog = true;
		},

		deleteItem(item) {
			const index = this.desserts.indexOf(item);
			confirm('Are you sure you want to delete this item?') && this.desserts.splice(index, 1);
		},

		close() {
			this.dialog = false;
			this.limpiar();
		},
		limpiar() {
			this.id = "";
			this.nombre = "";
			this.descripcion = "";
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
					.put('api/Categorias/Actualizar', {
						idcategoria: me.id,
						nombre: me.nombre,
						descripcion: me.descripcion,
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
					.post('api/Categorias/Crear', {
						nombre: me.nombre,
						descripcion: me.descripcion,
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
			if (this.nombre.length < 3 || this.nombre.length > 50) {
				this.validaMensaje.push('El nombre deber tener más de 3 caracteres y menos de 50 caracteres');
			}
			if (this.validaMensaje.length) {
				this.valida = 1;
			}
			return this.valida;
		},
		activarDesactivarMostrar(accion, item) {
			this.adModal = 1;
			this.adNombre = item.nombre;
			this.adId = item.idcategoria;
			if (accion == 1) {
				this.adAccion = 1;
			} else if (accion == 2) {
				this.adAccion = 2;
			} else {
				this.adAccion = 0;
			}
		},
		activarDesactivarCerrar(){
			this.adModal=0;
		},
		activar() {
			let me = this;
			axios
				.put('api/Categorias/Activar/' + this.adId, {})
				.then(function(response) {
					me.adModal=0;
					me.adAccion=0;
					me.adNombre="";
					me.adId="";
					me.listar();
				
				})
				.catch(function(error) {
					console.log(error);
				});
		},
		desactivar() {
			let me = this;
			axios
				.put('api/Categorias/Desactivar/' + this.adId, {})
				.then(function(response) {
					me.adModal=0;
					me.adAccion=0;
					me.adNombre="";
					me.adId="";
					me.listar();
				
				})
				.catch(function(error) {
					console.log(error);
				});
		},
		getColor(estado) {
			if (estado) return 'green';
			else return 'red';
		},
	},
};
</script>
