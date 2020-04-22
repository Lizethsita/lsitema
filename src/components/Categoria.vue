<template>
	<v-layout align-start>
		<v-flex>
			<v-data-table :headers="headers" :items="categorias" :search="search" sort-by="calories" class="elevation-1">
				<template v-slot:top>
					<v-toolbar flat color="white">
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
								<v-btn color="primary" dark class="mb-2" v-on="on">New Item</v-btn>
							</template>
							<v-card>
								<v-card-title>
									<span class="headline">{{ formTitle }}</span>
								</v-card-title>

								<v-card-text>
									<v-container>
										<v-row>
											<v-col cols="12" sm="6" md="4">
												<v-text-field
													v-model="editedItem.nombre"
													label="Nombre"
												></v-text-field>
											</v-col>
											<v-col cols="12" sm="6" md="4">
												<v-text-field
													v-model="editedItem.descripcion"
													label="Descripcion"
												></v-text-field>
											</v-col>
											
											<v-col cols="12" sm="6" md="4">
												<div v-if="true">  
													<v-text-field class="blue-text" label="Estado">Activo</v-text-field>
													
												</div>
												<div v-else>  
													<v-text-field class="red--text" label="Estado">Inactivo</v-text-field>
												</div>												
											</v-col>
											
										</v-row>
									</v-container>
								</v-card-text>

								<v-card-actions>
									<v-spacer></v-spacer>
									<v-btn color="blue darken-1" text @click="close">Cancelar</v-btn>
									<v-btn color="blue darken-1" text @click="save">Guardar</v-btn>
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
					<v-icon small @click="deleteItem(item)">
						delete
					</v-icon>
				</template>
				<template v-slot:no-data>
					<v-btn color="primary" @click="initialize">Resetear</v-btn>
				</template>
			</v-data-table>
		</v-flex>
	</v-layout>
</template>

<script>
import axios from 'axios'
export default {
	data() {
		return {
			categorias:[], 
			dialog: false,
			headers: [
				
				{ text: 'Nombre', value: 'nombre' },
				{ text: 'Descripción', value: 'descripcion', sortable: false },
				{ text: 'Estado', value: 'condicion', sortable: false },
				{ text: 'Opciones', value: 'opciones', sortable: false },
				
			],
			search: '',
			desserts: [],
			editedIndex: -1,
			editedItem: {
				name: '',
				calories: 0,
				fat: 0,
				carbs: 0,
				protein: 0
			},
			defaultItem: {
				name: '',
				calories: 0,
				fat: 0,
				carbs: 0,
				protein: 0
			}
		};
	},

	computed: {
		formTitle() {
			return this.editedIndex === -1 ? 'New Item' : 'Edit Item';
		}
	},

	watch: {
		dialog(val) {
			val || this.close();
		}
	},

	created() {
		console.log('inicio');
		this.initialize();
		this.listar();
	},

	methods: {
		listar(){
			let me=this;
			axios.get('api/Categorias/Listar').then(function(response){
				console.log(response);
				me.categorias = response.data;

			}).catch(function(error){
				console.log("error");

			});
		},
		initialize() {
			this.desserts = [
				{
					name: 'Frozen Yogurt',
					calories: 159,
					fat: 6.0,
					carbs: 24,
					protein: 4.0
				},
				{
					name: 'Ice cream sandwich',
					calories: 237,
					fat: 9.0,
					carbs: 37,
					protein: 4.3
				},
				{
					name: 'Eclair',
					calories: 262,
					fat: 16.0,
					carbs: 23,
					protein: 6.0
				},
				{
					name: 'Cupcake',
					calories: 305,
					fat: 3.7,
					carbs: 67,
					protein: 4.3
				},
				{
					name: 'Gingerbread',
					calories: 356,
					fat: 16.0,
					carbs: 49,
					protein: 3.9
				},
				{
					name: 'Jelly bean',
					calories: 375,
					fat: 0.0,
					carbs: 94,
					protein: 0.0
				},
				{
					name: 'Lollipop',
					calories: 392,
					fat: 0.2,
					carbs: 98,
					protein: 0
				},
				{
					name: 'Honeycomb',
					calories: 408,
					fat: 3.2,
					carbs: 87,
					protein: 6.5
				},
				{
					name: 'Donut',
					calories: 452,
					fat: 25.0,
					carbs: 51,
					protein: 4.9
				},
				{
					name: 'KitKat',
					calories: 518,
					fat: 26.0,
					carbs: 65,
					protein: 7
				}
			];
		},

		editItem(item) {
			this.editedIndex = this.desserts.indexOf(item);
			this.editedItem = Object.assign({}, item);
			this.dialog = true;
		},

		deleteItem(item) {
			const index = this.desserts.indexOf(item);
			confirm('Are you sure you want to delete this item?') && this.desserts.splice(index, 1);
		},

		close() {
			this.dialog = false;
			setTimeout(() => {
				this.editedItem = Object.assign({}, this.defaultItem);
				this.editedIndex = -1;
			}, 300);
		},

		save() {
			if (this.editedIndex > -1) {
				Object.assign(this.desserts[this.editedIndex], this.editedItem);
			} else {
				this.desserts.push(this.editedItem);
			}
			this.close();
		},
		getColor (estado) {
			if (estado) return 'green'
			else return 'red'
		}
	}
};
</script>
