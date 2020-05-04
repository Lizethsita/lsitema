<template>
	<v-layout align-start>
		<v-flex>
			<v-data-table
				:headers="headers"
				:items="roles"
				:search="search"
				sort-by="calories"
				class="elevation-1"
			>
				<template v-slot:top>
					<v-toolbar text color="white">
						<v-toolbar-title>Roles</v-toolbar-title>
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
			roles: [],
			dialog: false,
			headers: [
				{ text: 'Nombre', value: 'nombre' },
				{ text: 'Descripción', value: 'descripcion', sortable: false },
				{ text: 'Estado', value: 'condicion', sortable: false }
			],
			search: ''
		};
	},

	computed: {

	},

	watch: {

	},

	created() {
		console.log('inicio');
		this.listar();
	},

	methods: {
		listar() {
			let me = this;
			axios
				.get('api/Roles/Listar')
				.then(function(response) {
					console.log(response);
					me.roles = response.data;
				})
				.catch(function(error) {
					console.log('error');
				});
		},
		getColor(estado) {
			if (estado) return 'green';
			else return 'red';
		},
	},
};
</script>
