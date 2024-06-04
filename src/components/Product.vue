<template>
  <div class="container">
    <h1 class="mt-3 text-center">Pokémon Data</h1>

    <!-- Campo de búsqueda -->
    <div class="input-group mb-3">
      <input type="text" class="form-control" v-model="searchQuery" placeholder="Buscar Pokémon por nombre o ID" @keyup.enter="searchPokemon" />
      <button class="btn btn-primary" @click="searchPokemon">Buscar</button>
    </div>

    <!-- Mostrar mensaje de error -->
    <div v-if="errorMessage" class="alert alert-danger" role="alert">
      {{ errorMessage }}
    </div>

    <div class="row mt-3">
      <!-- Mostrar Pokémon encontrado -->
      <div v-if="foundPokemon" class="col-sm-12 mb-4">
        <div class="card h-100">
          <img :src="foundPokemon.sprites.front_default" class="card-img-top" alt="pokemon" @click="openModal(foundPokemon)" />
          <div class="card-body d-flex flex-column">
            <h5 class="card-title mb-2">{{ foundPokemon.name }}</h5>
            <p class="card-text text-center fw-bold">
              Abilities:
              <ul>
                <li v-for="ability in foundPokemon.abilities" :key="ability.ability.name">
                  {{ ability.ability.name }}
                </li>
              </ul>
            </p>
            <div class="mt-auto">
              <button class="btn btn-dark w-100" @click="openModal(foundPokemon)">
                Ver habilidades
              </button>
            </div>
          </div>
        </div>
      </div>

      <!-- Mostrar lista de Pokémon (desaparece si se ha encontrado uno) -->
      <div v-else v-for="pokemon in pokemonData" :key="pokemon.id" class="col-sm-6 col-md-4 col-lg-3 mb-4">
        <div class="card h-100">
          <img :src="pokemon.sprites.front_default" class="card-img-top" alt="pokemon" @click="openModal(pokemon)" />
          <div class="card-body d-flex flex-column">
            <h5 class="card-title mb-2">{{ pokemon.name }}</h5>
            <p class="card-text text-center fw-bold">
              Abilities:
              <ul>
                <li v-for="ability in pokemon.abilities" :key="ability.ability.name">
                  {{ ability.ability.name }}
                </li>
              </ul>
            </p>
            <div class="mt-auto">
              <button class="btn btn-dark w-100" @click="openModal(pokemon)">
                Ver habilidades
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>

    <!-- Modal -->
    <div class="modal fade" id="staticBackdrop" data-bs-backdrop="static" data-bs-keyboard="false" tabindex="-1" aria-labelledby="staticBackdropLabel" aria-hidden="true">
      <div class="modal-dialog">
        <div class="modal-content">
          <div class="modal-header">
            <h5 class="modal-title" id="staticBackdropLabel">{{ selectProduct ? selectProduct.name : '' }}</h5>
            <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
          </div>
          <div class="modal-body">
            <p>ID: {{ selectProduct ? selectProduct.id : '' }}</p>
            <p>Habilidades:</p>
            <ul>
              <li v-for="ability in (selectProduct ? selectProduct.abilities : [])" :key="ability.ability.name">
                {{ ability.ability.name }}
              </li>
            </ul>
            <img v-if="selectProduct" :src="selectProduct.sprites.front_default" alt="pokemon image" />
          </div>
          <div class="modal-footer">
            <button type="button" class="btn btn-secondary" data-bs-dismiss="modal">Close</button>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios';
import bootstrap from 'bootstrap/dist/js/bootstrap.bundle.min.js';
import { ref, nextTick } from 'vue';
import { pokeapi } from '@/composables/pokeApi'; // Asegúrate de que la ruta sea correcta

export default {
  name: 'App',
  data() {
    return {
      fields: ['abilities', 'id', 'name', 'sprites'],
      pokemonData: [],
      selectProduct: null, // Estado para el Pokémon seleccionado
      searchQuery: '', // Campo de búsqueda
      foundPokemon: null, // Estado para el Pokémon encontrado
      errorMessage: '', // Mensaje de error
    };
  },
  mounted() {
    this.fetchData();
  },
  methods: {
    fetchData() {
      axios.get(pokeapi)
        .then(response => {
          if (response.data && response.data.results) {
            const requests = response.data.results.map(pokemon => axios.get(pokemon.url));
            Promise.all(requests)
              .then(responses => {
                this.pokemonData = responses.map(response => response.data);
                console.log(this.pokemonData);
              })
              .catch(error => {
                console.error('Error fetching individual Pokemon data:', error);
              });
          } else {
            console.error('Unexpected response structure:', response);
          }
        })
        .catch(error => {
          console.error('Error fetching Pokemon list:', error);
        })
        .finally(() => {
          console.log('Request completed');
        });
    },
    searchPokemon() {
      if (this.searchQuery.trim() !== '') {
        const searchUrl = `https://pokeapi.co/api/v2/pokemon/${this.searchQuery.toLowerCase()}`;
        axios.get(searchUrl)
          .then(response => {
            this.foundPokemon = response.data;
            this.errorMessage = ''; // Limpiar mensaje de error
            console.log('Found Pokemon:', this.foundPokemon);
          })
          .catch(error => {
            console.error('Error fetching Pokemon:', error);
            this.foundPokemon = null;
            this.errorMessage = 'No se encontró el Pokémon. Por favor, inténtalo de nuevo.'; // Mostrar mensaje de error
          });
      }
    },
    openModal(pokemon) {
      this.selectProduct = pokemon;
      nextTick(() => {
        const modalElement = document.getElementById('staticBackdrop');
        const modalInstance = new bootstrap.Modal(modalElement);
        modalInstance.show();
      });
    },
  },
};
</script>

<style scoped>
img {
  padding: 0.5rem;
  width: 200px;
  height: 100%;
  object-fit: contain;
  margin: 0 auto;
  cursor: pointer;
}
</style>
