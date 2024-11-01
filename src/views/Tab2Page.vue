<template>
  <ion-page>
    <ion-header>
      <ion-toolbar>
        <ion-title>Tab 2</ion-title>
      </ion-toolbar>
    </ion-header>
    <ion-content :fullscreen="true">
      <ion-grid>
        <ion-row style="display: grid;">
          <ion-col>
            <ion-label>Lokasi Anda: {{ posisilat }}, {{ posisilong }}</ion-label>
          </ion-col>
          <ion-col>
            <ion-label>Posisi Kantor: {{ officelat }}, {{ officelong }}</ion-label>
          </ion-col>
          <ion-col>
            <ion-label>Jarak: {{ jarak }} km</ion-label>                        
          </ion-col>
          <ion-col>
            <ion-label>Status: {{ status }}</ion-label>            
          </ion-col>
        </ion-row>
      </ion-grid>

      <ion-fab vertical="bottom" horizontal="center" slot="fixed">
        <ion-fab-button @click="locatorButtonPressed">
          <ion-icon :icon="map"></ion-icon>
        </ion-fab-button>
      </ion-fab>
    </ion-content>
  </ion-page>
</template>

<script lang="ts">
import { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonLabel, IonGrid, IonRow, IonCol, IonFab, IonIcon } from '@ionic/vue';
import { map } from 'ionicons/icons';

export default {
    components: { IonPage, IonHeader, IonToolbar, IonTitle, IonContent, IonLabel, IonGrid, IonRow, IonCol, IonFab, IonIcon },
    data() {
        return {
            error: '',
            officelat: -6.3380311,
            officelong: 106.7417130,
            jarak: 0,
            status: '',
            posisilat: 0,
            posisilong: 0,
            map,
            locationWatchId: null, // Store watch ID
            updateIntervalId: null // Store interval ID
        };
    },
    methods: {
        locatorButtonPressed() {
            // Clear any existing location watch or interval to avoid duplicate
            if (this.locationWatchId) {
                navigator.geolocation.clearWatch(this.locationWatchId);
            }
            if (this.updateIntervalId) {
                clearInterval(this.updateIntervalId);
            }

            // Start watching location changes
            this.locationWatchId = navigator.geolocation.watchPosition(
                (position) => {
                    this.posisilat = position.coords.latitude;
                    this.posisilong = position.coords.longitude;
                },
                (error) => {
                    console.error("Geolocation error:", error.message);
                    this.error = error.message;
                }
            );

            // Set interval to calculate distance every 2 seconds
            this.updateIntervalId = setInterval(() => {
                this.calculateDistance();
            }, 2000); // Update every 2 seconds
        },

        calculateDistance() {
            this.jarak = this.haversineDistance(this.posisilat, this.posisilong, this.officelat, this.officelong);

            // Check if within radius
            const radius = 0.01; // Radius in km
            this.status = this.jarak <= radius ? "Anda di dalam radius" : "Anda di luar radius";

            console.log("Calculated Distance (km):", this.jarak);
        },

        // Haversine formula to calculate distance in meters
        haversineDistance(lat1, lon1, lat2, lon2) {
            const R = 6371; // Radius of Earth in meters
            const lat1Rad = lat1 * Math.PI / 180;
            const lat2Rad = lat2 * Math.PI / 180;
            const deltaLat = (lat2 - lat1) * Math.PI / 180;
            const deltaLon = (lon2 - lon1) * Math.PI / 180;

            const a = Math.sin(deltaLat / 2) * Math.sin(deltaLat / 2) +
                      Math.cos(lat1Rad) * Math.cos(lat2Rad) *
                      Math.sin(deltaLon / 2) * Math.sin(deltaLon / 2);

            const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1 - a));
            const d = R * c; // Distance in meters

            console.log("Haversine Calculation -> a:", a, "c:", c, "Distance (meters):", d);
            return d;
        }
    },
    beforeUnmount() {
        // Clear the watch and interval when component is unmounted
        if (this.locationWatchId) {
            navigator.geolocation.clearWatch(this.locationWatchId);
        }
        if (this.updateIntervalId) {
            clearInterval(this.updateIntervalId);
        }
    }
};
</script>
