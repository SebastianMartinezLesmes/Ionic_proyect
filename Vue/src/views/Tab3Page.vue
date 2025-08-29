<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Episodes of Rick and Morty</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-header collapse="condense">
        <ion-toolbar>
          <ion-title size="large">Episodes</ion-title>
        </ion-toolbar>
      </ion-header>

      <ion-accordion-group>
        <ion-accordion v-for="episode in episodes" :key="episode.id">
          <ion-item slot="header">
            <ion-label>{{ episode.name }}</ion-label>
          </ion-item>
          <div class="ion-padding" slot="content">
            <p id="text_out_time"><strong> Air Date:</strong> {{ episode.air_date }}</p>
            <p id="text_out_time2"><strong>Characters:</strong></p>
            <div id="item_contenedor">
              <ion-item lines="none" v-for="character in episode.characters" :key="character.id" id="list_characters">
                <img slot="start" :src="character.character_img" alt="Character Image" id="img">
                <ion-label slot="end">{{ character.character_name }}</ion-label>
              </ion-item>
            </div>
          </div>
        </ion-accordion>
      </ion-accordion-group>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { onMounted, ref } from 'vue';
import { 
  IonPage, 
  IonHeader, 
  IonToolbar, 
  IonTitle, 
  IonContent, 
  IonAccordionGroup, 
  IonAccordion, 
  IonLabel, 
  IonItem,
} from '@ionic/vue';
import '../theme/tab3Page.css';

const episodes = ref([]);

async function fetchAllEpisodes() {
  try {
    let page = 1;
    let allEpisodes = [];
    let fetchMore = true;

    while (fetchMore) {
      const response = await fetch(`https://rickandmortyapi.com/api/episode?page=${page}`);
      const data = await response.json();
      allEpisodes = [...allEpisodes, ...data.results];

      for (const episode of data.results) {
        episode.characters = await Promise.all(episode.characters.map(async characterUrl => {
          const characterResponse = await fetch(characterUrl);
          const characterData = await characterResponse.json();
          return {
            id: characterData.id,
            character_name: characterData.name,
            character_img: characterData.image
          };
        }));
      }

      if (data.info.next) {
        page++;
      } else {
        fetchMore = false;
      }
    }

    episodes.value = allEpisodes;
  } catch (error) {
    console.error('Error fetching episodes:', error);
  }
}

onMounted(() => {
  fetchAllEpisodes();
});
</script>
