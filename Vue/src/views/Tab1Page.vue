<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Characters of Rick and Morty</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div id="content">
        <ion-card id="arriba" v-if="selectedCharacter">
          <div id="ar_izquierda">
            <img :src="selectedCharacter?.image" alt="Character Image">
          </div>
          <div id="ar_derecha">
            <div>
              <span>{{ selectedCharacter.name }}</span>
              <p>Status: {{ selectedCharacter.status }}</p>
              <p>Species: {{ selectedCharacter.species }}</p>
              <p>Gender: {{ selectedCharacter.gender }}</p>
              <p>Origin: {{ selectedCharacter.origin.name }}</p>
            </div>
          </div>
          <ion-button id="ar_button" @click="clearSelectedCharacter">X</ion-button>
        </ion-card>

        <div id="search_bar">
          <ion-searchbar 
            v-model="searchQuery" 
            placeholder="Search for a character" 
            color="success"
            @ionInput="filterCharacters">
          </ion-searchbar>
        </div>

        <div id="abajo">
          <div v-for="character in filteredCharacters" :key="character.id" @click="selectCharacter(character)">
            <ion-card id="ab_card">
              <img :src="character.image" alt="Character Image"/>
              <ion-header>{{character.name}}</ion-header>
            </ion-card>
          </div>
        </div>
        
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted, computed } from 'vue';
import { 
  IonPage, 
  IonHeader, 
  IonToolbar, 
  IonTitle, 
  IonContent, 
  IonCard, 
  IonSearchbar,
  IonButton, 
} from '@ionic/vue';
import '../theme/tab1Page.css';

const characters = ref([]);
const selectedCharacter = ref(null);
const searchQuery = ref('');

async function fetchAllCharacters() {
  try {
    let page = 1;
    let allCharacters = [];
    let fetchMore = true;

    while (fetchMore) {
      const response = await fetch(`https://rickandmortyapi.com/api/character?page=${page}`);
      const data = await response.json();
      allCharacters = [...allCharacters, ...data.results];

      if (data.info.next) {
        page++;
      } else {
        fetchMore = false;
      }
    }

    characters.value = allCharacters;
  } catch (error) {
    console.error('Error fetching characters:', error);
  }
}

function selectCharacter(character) {
  selectedCharacter.value = character;
}

function clearSelectedCharacter() {
  selectedCharacter.value = null;
}

const filteredCharacters = computed(() => {
  if (!searchQuery.value) {
    return characters.value;
  }
  return characters.value.filter(character =>
    character.name.toLowerCase().includes(searchQuery.value.toLowerCase())
  );
});

onMounted(() => {
  fetchAllCharacters();
});
</script>
