<script lang="ts" setup>
import { ref } from 'vue';
import { useFetch } from '#app'; // Si usas Nuxt

export interface Character {
  info: Info;
  results: Result[];
}

export interface Info {
  count: number;
  pages: number;
  next: string;
  prev: null;
}

export interface Result {
  id: number;
  name: string;
  status: string;
  species: string;
  type: string;
  gender: string;
  image: string;
}

const { data } = await useFetch<Character>('https://rickandmortyapi.com/api/character');

// Convertimos los resultados a una lista reactiva para poder manipularlos
const characters = ref<Result[]>(data.value?.results || []);

// Estado para controlar el diálogo del formulario
const dialog = ref(false);
const editing = ref(false);
const currentCharacter = ref<Result | null>(null);

// Función para abrir el formulario con un personaje seleccionado
const openDialog = (character: Result | null = null) => {
  currentCharacter.value = character
    ? { ...character } // Clonar para evitar modificar el original hasta confirmar
    : { id: Date.now(), name: '', status: '', species: '', type: '', gender: '', image: '' };
  editing.value = !!character;
  dialog.value = true;
};

// Guardar personaje (añadir o editar)
const saveCharacter = () => {
  if (!currentCharacter.value) return;

  if (editing.value) {
    // Editar personaje existente
    const index = characters.value.findIndex((c) => c.id === currentCharacter.value?.id);
    if (index !== -1) {
      characters.value[index] = { ...currentCharacter.value };
    }
  } else {
    // Agregar nuevo personaje
    characters.value.push({ ...currentCharacter.value });
  }

  dialog.value = false;
};

// Eliminar personaje
const deleteCharacter = (id: number) => {
  characters.value = characters.value.filter((c) => c.id !== id);
};
</script>

<template>
  <v-container fluid>
    <v-row>
      <v-col v-for="item in characters" :key="item.id" cols="12" md="4">
        <v-card class="mx-auto" max-width="344">
          <v-img height="200px" :src="item.image || 'https://via.placeholder.com/200'" cover></v-img>

          <v-card-title>{{ item.name }}</v-card-title>

          <v-card-subtitle>{{ item.species }} - {{ item.status }}</v-card-subtitle>

          <v-card-actions>
            <v-btn color="blue" @click="openDialog(item)">Editar</v-btn>
            <v-btn color="red" @click="deleteCharacter(item.id)">Eliminar</v-btn>
          </v-card-actions>
        </v-card>
      </v-col>
    </v-row>

    <!-- Botón para agregar un nuevo personaje -->
    <v-btn color="green" class="mt-4" @click="openDialog()">Agregar Personaje</v-btn>

    <!-- Diálogo de formulario -->
    <v-dialog v-model="dialog" persistent max-width="500px">
      <v-card>
        <v-card-title>
          <span class="text-h5">{{ editing ? 'Editar' : 'Agregar' }} Personaje</span>
        </v-card-title>

        <v-card-text>
          <v-container>
            <v-row>
              <v-col cols="12">
                <v-text-field label="Nombre" v-model="currentCharacter!.name" required></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field label="Estatus" v-model="currentCharacter!.status"></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field label="Especie" v-model="currentCharacter!.species"></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field label="Género" v-model="currentCharacter!.gender"></v-text-field>
              </v-col>
              <v-col cols="12">
                <v-text-field label="Imagen URL" v-model="currentCharacter!.image"></v-text-field>
              </v-col>
            </v-row>
          </v-container>
        </v-card-text>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="red" @click="dialog = false">Cancelar</v-btn>
          <v-btn color="green" @click="saveCharacter">Guardar</v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-container>
</template>
