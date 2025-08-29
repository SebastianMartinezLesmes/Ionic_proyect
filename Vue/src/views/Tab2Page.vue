<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Locations of Rick and Morty</ion-title>
      </ion-toolbar>
    </ion-header>

    <ion-content :fullscreen="true">
      <div id="content_page">
        <div id="izquierda">
          <ion-accordion-group>
            <ion-accordion v-for="dimension in dimensions" :key="dimension.dimension" >
              <ion-item slot="header" id="acordeon_header">
                <ion-label>{{ dimension.dimension }}</ion-label>
              </ion-item>
              <ion-list slot="content" id="acordeon_content">
                <ion-item lines="none" v-for="location in dimension.locations" :key="location.id" id="acordeon_label">
                  <ion-label @click="getDatesLocation(location)">
                    <h2>{{ location.name }}</h2>
                  </ion-label>
                </ion-item>
              </ion-list>
            </ion-accordion>
          </ion-accordion-group>
        </div>
        <div id="derecha" v-if="LocationSelected">
          <ion-button id="button_Clear_residents" @click="clear()"> <h2> Close </h2> </ion-button>
          <h2>{{ LocationSelected.name }}</h2>
          <h2>Type: {{ LocationSelected.type }}</h2>
          <h2>Dimension: {{ LocationSelected.dimension }}</h2>
          <h3>Residents:</h3>
          <ul id="list_residents">
            <ion-card v-for="resident in LocationSelected.residentsDetails" :key="resident.id">
              <img :src="resident.image" alt="Resident image" width="50">
              <h2>{{ resident.name }}</h2>
            </ion-card>
          </ul>
        </div>
      </div>
    </ion-content>
  </ion-page>
</template>

<script setup lang="ts">
import { ref, onMounted } from 'vue';
import { 
  IonPage, 
  IonHeader, 
  IonToolbar, 
  IonTitle, 
  IonContent, 
  IonAccordion, 
  IonAccordionGroup, 
  IonLabel, 
  IonItem,
  IonList,
  IonCard,
} from '@ionic/vue';
import '../theme/tab2Page.css';

interface Resident {
  id: number;
  name: string;
  image: string;
}

interface Location {
  id: number;
  name: string;
  type: string;
  dimension: string;
  residents: string[];
  residentsDetails: Resident[]; // New field for resident details
}

interface Dimension {
  dimension: string;
  locations: Location[];
}

const dimensions = ref<Dimension[]>([]);
const LocationSelected = ref<Location | null>(null);

async function fetchLocations() {
  try {
    let page = 1;
    let fetchMore = true;
    const allLocations: Location[] = [];

    while (fetchMore) {
      const response = await fetch(`https://rickandmortyapi.com/api/location?page=${page}`);
      const data = await response.json();
      
      const locations = data.results.map((loc: any): Location => ({
        id: loc.id,
        name: loc.name,
        type: loc.type,
        dimension: loc.dimension,
        residents: loc.residents,
        residentsDetails: [] // Initialize with empty array
      }));
      allLocations.push(...locations);
      
      if (data.info.next) {
        page++;
      } else {
        fetchMore = false;
      }
    }

    const dimensionsMap = new Map<string, Dimension>();

    allLocations.forEach(location => {
      const dimension = location.dimension;
      if (!dimensionsMap.has(dimension)) {
        dimensionsMap.set(dimension, {
          dimension: dimension,
          locations: []
        });
      }
      dimensionsMap.get(dimension)!.locations.push(location);
    });

    dimensions.value = Array.from(dimensionsMap.values());
  } catch (error) {
    console.error('Error fetching locations:', error);
  }
}

async function getDatesLocation(location: Location) {
  LocationSelected.value = location;
  console.log(LocationSelected.value);

  try {
    const residentsDetails: Resident[] = await Promise.all(
      location.residents.map(async (residentUrl) => {
        const response = await fetch(residentUrl);
        const data = await response.json();
        return {
          id: data.id,
          name: data.name,
          image: data.image
        };
      })
    );

    LocationSelected.value = { ...LocationSelected.value, residentsDetails };
  } catch (error) {
    console.error('Error fetching resident details:', error);
  }
}

async function clear() {
  LocationSelected.value = null;
}

onMounted(() => {
  fetchLocations();
});
</script>

