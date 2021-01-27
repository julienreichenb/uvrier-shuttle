<template>
  <div style="width: 100%">
    <l-map
      style="height: 82vh"
      v-if="showMap"
      :zoom="zoom"
      :center="center"
      :options="mapOptions"
      @update:center="centerUpdate"
      @update:zoom="zoomUpdate"
    >
      <l-tile-layer
        :url="url"
        :attribution="attribution"
      />
      <l-control position="topleft">
        <b-button class="m-2 p-0 bg-transparent border-0" v-b-modal="'info-modal'">
        <font-awesome-layers class="fa-7x">
          <font-awesome-icon icon="circle" class="text-primary" />
          <font-awesome-icon icon="circle" transform="shrink-2" />
          <font-awesome-icon icon="info" class="text-primary" transform="shrink-6"  />
        </font-awesome-layers>
        </b-button>
      </l-control>
       <l-control position="bottomright">
        <b-button variant="success" @click="showBook = true" class="border-white p-4">
          <font-awesome-layers class="fa-5x align-bottom mr-4">
            <font-awesome-icon icon="circle" />
            <font-awesome-icon icon="bookmark" class="text-success" transform="shrink-7"  />
          </font-awesome-layers>
          <span class="display-3">Réserver un trajet</span>
        </b-button>
      </l-control>
      <l-control position="topright">
        <div class="bg-white border border-dark">
          <b-row no-gutters class="text-success border-bottom border-secondary" style="padding: 10px; width: 400px;">
            <b-col sm="3" class="my-auto">
                <font-awesome-layers class="fa-4x">
                  <font-awesome-icon icon="bus" color="black" />
                  <font-awesome-layers class="ml-4 mt-3" >
                      <font-awesome-icon icon="circle" color="black" transform="shrink-4" />
                      <font-awesome-icon icon="circle" color="white" transform="shrink-5" />
                      <font-awesome-icon icon="check" transform="shrink-8" />
                  </font-awesome-layers>
              </font-awesome-layers>
            </b-col>
            <b-col class="text-left pl-4">
              <h4 class="pt-2">Navette(s) en service</h4>
              <h2>{{ shuttleFree }}</h2>
            </b-col>
          </b-row>
          <!--
          <b-row no-gutters class="text-danger border-bottom border-secondary" style="padding: 10px; width: 400px;">
            <b-col sm="3" class="my-auto">
              <font-awesome-layers class="fa-4x">
                <font-awesome-icon icon="bus" color="black" />
                  <font-awesome-layers class="ml-4 mt-3" >
                    <font-awesome-icon icon="circle" color="black" transform="shrink-4" />
                    <font-awesome-icon icon="circle" color="white" transform="shrink-5" />
                    <font-awesome-icon icon="times" transform="shrink-8" />
                  </font-awesome-layers>              
                </font-awesome-layers>
            </b-col>
            <b-col class="text-left pl-4">
              <h4 class="pt-2">Véhicules occupés</h4>
              <h2>{{ shuttleBusy }}</h2>
            </b-col>
          </b-row>
          -->
          <b-row no-gutters class="text-info" style="padding: 10px; width: 400px;">
            <b-col sm="3" class="my-auto">
              <font-awesome-icon icon="clock" class="fa-4x" />
            </b-col>
            <b-col class="text-left pl-4">
              <h4 class="pt-2">Attente moyenne</h4>
              <h2>{{ avgWaitingTime }} minute(s)</h2>
            </b-col>
          </b-row>
        </div>
        <b-card class="mt-3 border border-dark" header-bg-variant="primary" header-text-variant="white">
          <template v-slot:header>
            <h3 class="mt-1">Annonces</h3>
          </template>
          <h4 class="text-left">{{ liveInfo }}</h4>
        </b-card>
      </l-control>
      <l-circle-marker v-if="origin" 
        :lat-lng="origin.location"
        :radius="circle.radius"
        :color="originColor"
        :fill-color="originColor"
        :fill-opacity="circle.fillOpacity"
        :weight="circle.weight">
        <l-tooltip :content="origin.name" :options="{ permanent: true, direction: 'auto' }"/>
      </l-circle-marker>
      <l-circle-marker 
        v-for="(destination, index) in destinations" 
        :key="'destination-' + index" 
        :lat-lng="destination.location"
        :radius="circle.radius"
        :color="destinationColor"
        :fill-color="destinationColor"
        :fill-opacity="circle.fillOpacity"
        :weight="circle.weight">
        <l-tooltip :content="destination.name" :options="{ permanent: true, direction: 'auto' }"/>
      </l-circle-marker>
      <l-marker 
        v-for="shuttle in shuttlePositions" 
        :key="shuttle.id" 
        :lat-lng="shuttle.location">
        <l-icon>
          <font-awesome-layers>
            <font-awesome-icon icon="circle" class="fa-3x" />
            <font-awesome-icon icon="circle" class="fa-3x text-white" transform="shrink-1" />
            <font-awesome-icon icon="bus" class="fa-3x text-success" transform="shrink-6" />
          </font-awesome-layers>
        </l-icon>
      </l-marker>
    </l-map>
    <InfoModal />
    <BookModal :show="showBook" @booking="confirmBooking" @hide="hideBooking" />
    <ConfirmModal :booking="booked" :show="showConfirm" @reset="reset" />
  </div>
</template>

<script>
import { latLng } from 'leaflet'
import InfoModal from './InfoModal'
import BookModal from './BookModal'
import ConfirmModal from './ConfirmModal'
import { API_KEY, API_SERVICE_ID, API_SERVER, ORIGIN_FIND_NAME } from '../constants'
export default {
  components: {
  InfoModal,
  BookModal,
  ConfirmModal,
  },
  mounted() {
    // Initial load
    this.loadShuttlesPositions()
    this.loadOriginAndDestinations()
    // Reoad Shuttles' position every 5 seconds
    setInterval(() => {
      this.loadShuttlesPositions()
    }, 5000);
  },
  data() {
    return {
      origin: null,
      destinations: null,
      shuttleNumber: 2,
      zoom: 16,
      center: latLng(46.2507967, 7.4220283),
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
      '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      currentZoom: 11.5,
      currentCenter: latLng(46.2513967, 7.4136283),
      showParagraph: false,
      mapOptions: {
        zoomSnap: 0.5,
        zoomControl: false,
      },
      showMap: true,      
      shuttlePositions: null,
      showBook: false,
      showConfirm: false,
      booked: null,
      shuttleFree: 0,
      shuttleBusy: 0,
      avgWaitingTime: 0,
      liveInfo: 'Traffic régulier.',
      circle: {
        radius: 4,
        fillOpacity: 0.5,
        weight: 2
      },
      originColor: 'green',
      destinationColor: 'red'
    }
  },
  methods: {
    zoomUpdate(zoom) {
      this.currentZoom = zoom
    },
    centerUpdate(center) {
      this.currentCenter = center
    },
    loadShuttlesPositions() {
      // Call API to get the current shuttles'position      
      fetch(API_SERVER + '/transportation/v1/vehicles', {
        method: 'GET',
        headers: {
          'Accept': 'application/json',
          'apiKey': API_KEY
        }
      })
      .then(response => { 
          if(response.ok){
              return response.json()    
          } else{
              this.shuttlePositions = null
              this.shuttleFree = 0
              this.shuttleBusy = 0;
              alert("Server returned " + response.status + " : " + response.statusText);
          }                
      })
      .then(response => {
          if(response){
              this.shuttlePositions = response
              this.shuttleFree = this.shuttlePositions.length
              this.shuttleBusy = this.shuttleNumber - this.shuttlePositions.length;
          }
      })
      .catch(err => {
          console.log(err);
      });
      
      this.avgWaitingTime = 2      
    },
    loadOriginAndDestinations() {
      // Call API to get the stops position      
      fetch(API_SERVER + '/transportation/v1/services/' + API_SERVICE_ID + '/stops', {
        method: 'GET',
        headers: {
          'Accept': 'application/json',
          'apiKey': API_KEY
        }
      })
      .then(response => { 
          if(response.ok){
              return response.json()    
          } else{
              this.origin
              this.destinations = null
              alert("Server returned " + response.status + " : " + response.statusText);
          }                
      })
      .then(response => {
          if(response){
              const stops = response
              this.origin = stops.find(s => s.name.toLowerCase().includes(ORIGIN_FIND_NAME))                    
              this.destinations = stops.filter(s => s.id !== this.origin.id)
          }
      })
      .catch(err => {
          console.log(err);
      });
      
      this.avgWaitingTime = 2      
    },
    confirmBooking(booked) {
      this.booked = booked
      this.showConfirm = true
      this.showBook = false
      // Close confirmation after 5 seconds
      setTimeout(() => {
        this.reset()
      }, 5000)
    },
    hideBooking() {
      this.showBook = false
    },
    reset() {
      this.booked = null
      this.showConfirm = false
      this.showBook = false
    },
  },
}
</script>

<style>
.icon-shuttle {
  background-color: red ;
}
</style>