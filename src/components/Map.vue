<template>
  <div style="width: 100%">
    <l-map
      style="height: calc(100vh - 205px)"
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
        <b-button v-if="serviceAvailable" variant="success" @click="$bvModal.show('book-modal')" class="border-white p-4">
          <font-awesome-layers class="fa-5x align-bottom mr-4">
            <font-awesome-icon icon="circle" />
            <font-awesome-icon icon="tablet-alt" class="text-success" transform="shrink-5" />
            <font-awesome-icon icon="bullseye" class="text-success" transform="shrink-11 up-0.5 left-0.25" />
            <font-awesome-icon icon="hand-point-up" class="text-white" transform="shrink-8 down-2 right-2.2 rotate--30"/>
            <font-awesome-icon icon="hand-point-up" class="text-success" transform="shrink-9 down-2 right-2.2 rotate--30"/>            
          </font-awesome-layers>
          <span class="display-3">Réserver un trajet</span>
        </b-button>
      </l-control>
      <l-control position="topright">
        <div class="bg-white border border-dark">
          <b-row v-if="serviceAvailable" no-gutters class="text-success border-bottom border-secondary" style="padding: 10px; width: 400px;">
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
          <b-row v-if="!serviceAvailable" no-gutters class="text-danger border-bottom border-secondary" style="padding: 10px; width: 400px;">
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
              <h4 class="pt-2">Aucun véhicule disponible</h4>              
            </b-col>
          </b-row>          
          <b-row v-if="serviceAvailable" no-gutters class="text-info" style="padding: 10px; width: 400px;">
            <b-col sm="3" class="my-auto">
              <font-awesome-icon icon="clock" class="fa-4x" />
            </b-col>
            <b-col class="text-left pl-4">
              <h4 class="pt-2">Attente moyenne</h4>
              <h2 v-if="avgWaitingTime > 1">{{ avgWaitingTime }} minutes</h2>
              <h2 v-else>{{ avgWaitingTime }} minute</h2>
            </b-col>
          </b-row>
        </div>
        <b-card v-if="announcements && announcements.length" class="mt-3 border border-dark" header-bg-variant="primary" header-text-variant="white" style="width:400px;">
          <template v-slot:header>
            <h3 class="mt-1">Annonces</h3>
          </template>
          <div class="announcements-list">
            <div v-for="(announcement, i) in announcements" :key="'announcement-' + i">
              <div v-for="contentlanguage in announcement.contentByLanguage" :key="'contentlanguage-' + i + '-' + contentlanguage.language.languageCode">
                <h4>{{ contentlanguage.content.title }}</h4>
                <p class="lead"><strong>{{ contentlanguage.content.text }}</strong></p>
              </div>
            </div>
          </div>          
        </b-card>
      </l-control>
      <l-marker v-if="origin" :lat-lng="origin.location" :icon="getIcon(originColor)">
        <l-tooltip :content="origin.name" :options="{ permanent: true, direction: 'auto' }"/>
      </l-marker>
      <l-circle-marker 
        v-for="(destination, index) in destinations" 
        :key="'destination-' + index" 
        :lat-lng="destination.location"
        :radius="circleMarker.radius"
        :color="destinationColor"
        :fill-color="destinationColor"
        :fill-opacity="circleMarker.fillOpacity"
        :weight="circleMarker.weight">
        <l-tooltip v-if="customDestinations.includes(destination.name.toLowerCase())" :content="destination.name" :options="{ permanent: true, direction: 'left' }"/>
        <l-tooltip v-else :content="destination.name" :options="{ permanent: true, direction: 'right' }"/>
      </l-circle-marker>
      <l-marker 
        v-for="shuttle in shuttlePositions" 
        :key="shuttle.id" 
        :lat-lng="shuttle.location">
        <l-icon :icon-anchor="[18,6]">
          <font-awesome-layers>
            <font-awesome-icon icon="circle" class="fa-3x text-white" />
            <font-awesome-icon icon="circle" class="fa-3x text-success" transform="shrink-1" />
            <font-awesome-icon icon="bus" class="fa-3x text-white" transform="shrink-7" />
            <font-awesome-layers-text counter :value="shuttle.name" class="fa-4x badge-shuttle" position="top-left" />
          </font-awesome-layers>          
        </l-icon>
      </l-marker>
    </l-map>
    <InfoModal />
    <BookModal v-if="origin && destinations" @booking="confirmBooking" :destinations="destinations" :start="origin"/>
    <ConfirmModal v-if="confirmation" :confirmation="confirmation" :show="showConfirm" @close="close"/>
  </div>
</template>

<script>
import { latLng, divIcon } from 'leaflet'
import InfoModal from './InfoModal'
import BookModal from './BookModal'
import ConfirmModal from './ConfirmModal'
import moment from 'moment'
export default {
  components: {
  InfoModal,
  BookModal,
  ConfirmModal,
  },
  async mounted() {    
    // Load Shuttles' position every 1 second
    setInterval(async () => {
      await this.loadShuttlesPositions()
    }, 1000);
    // Initial load
    await this.loadOriginAndDestinations()
    await this.computeAvgWaitingTime()
    await this.loadAnnouncements()
    // Calculate waiting time and load traveler announcements every 1 min    
    setInterval(async () => {
      await this.computeAvgWaitingTime()
      await this.loadAnnouncements()
    }, 60000);
  },  
  data() {
    return {
      origin: null,
      destinations: null,
      announcements: null,
      customDestinations: ['uvrier le puits 1', 'uvrier charmilles 2', 'uvrier jardin public', 'uvrier la plaine', 'uvrier les lucioles'],
      shuttleNumber: 2,
      zoom: 16,
      center: latLng(46.25055924910488, 7.417771589825862),
      url: 'https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png',
      attribution:
      '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      currentZoom: 11.5,
      currentCenter: latLng(46.25055924910488, 7.417771589825862),
      showParagraph: false,      
      mapOptions: {
        zoomSnap: 0.5,
        zoomControl: false,
        dragging: false,
        touchZoom: false,
        scrollWheelZoom: false,
        doubleClickZoom: false
      },
      showMap: true,      
      shuttlePositions: null,
      showConfirm: false,
      confirmation: null,
      shuttleFree: 0,
      serviceAvailable: false,
      avgWaitingTime: 0,
      circleMarker: {
        radius: 4,
        fillOpacity: 0.5,
        weight: 2
      },
      destinationColor: '#28a745',
      originColor: {
        color:'#c11a1a',
        strokeColor:'#d73534',
        circleColor:'#590000'
      }
    }
  },
  methods: {
    zoomUpdate(zoom) {
      this.currentZoom = zoom
    },
    centerUpdate(center) {
      this.currentCenter = center
    },
    async loadShuttlesPositions() {
      // Call API to get the current shuttles'position
      let vehicles = await this.getVehicles()
      if(vehicles) {
        this.shuttlePositions = vehicles
        this.shuttleFree = this.shuttlePositions.length
        if(this.shuttleFree > 0) {
          this.serviceAvailable = true
        } else {
          this.serviceAvailable = false
        }
      } else {
        this.shuttlePositions = null
        this.shuttleFree = 0
        this.serviceAvailable = false
      }      
    },
    async loadOriginAndDestinations() {
      // Call API to get the stops position
      let stops = await this.getStops()
      if(stops) {
        this.origin = stops.find(s => s.name.toLowerCase().includes(process.env.VUE_APP_ORIGIN_FIND_NAME))
        this.destinations = stops.filter(s => s.id !== this.origin.id).sort((a,b) => a.name.localeCompare(b.name))
      } else {
        this.origin = null
        this.destinations = null  
      }
    },
    async confirmBooking(confirmation) {
      this.confirmation = confirmation
      this.showConfirm = true
      this.$bvModal.hide('book-modal')
    },
    reset() {
      this.confirmation = null
      this.showConfirm = false
    },
    async computeAvgWaitingTime() {
        // Call API to get the max 10 last rides
        let response = await this.getRides()
        if(response.result.length > 0) {
          let waitingTimes = []
          for(let i=0; i< response.result.length; i++) {
            let start = moment(response.result[i].createdAt)
            let end = moment(response.result[i].pickupTime)
            let duration = moment.duration(end.diff(start)).minutes()
            waitingTimes.push(duration)
          }
          let sum = waitingTimes.reduce((a, b) => a + b, 0)          
          this.avgWaitingTime = (sum / response.result.length).toFixed(0)
        }
        else {
            this.avgWaitingTime = 0
        }        
    },
    async loadAnnouncements() {
      // Call API to get the active traveler announcements
      let announcements = await this.getAnnouncements()
      if(announcements) {
        this.announcements = announcements
      } else {
        this.announcements = null
      }
    },
    async getStops() {
      // Call API to get the stops      
      try {
        let response = await fetch(process.env.VUE_APP_API_SERVER + '/transportation/v1/services/' + process.env.VUE_APP_API_SERVICE_ID + '/stops', {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'apiKey': process.env.VUE_APP_API_KEY
          }          
        })

        if(response.ok) {
          return response.json()
        } else {
          console.log('Server returned ' + response.status + ' : ' + response.statusText);                    
        }
      } catch (error) {
        console.log(error);
      }
    },
    async getVehicles() {
      // Call API to get the vehicles
      try {
        let response = await fetch(process.env.VUE_APP_API_SERVER + '/transportation/v2/services/' + process.env.VUE_APP_API_SERVICE_ID + '/vehicles', {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'apiKey': process.env.VUE_APP_API_KEY
          }          
        })

        if(response.ok) {
          return response.json()
        } else {
          console.log('Server returned ' + response.status + ' : ' + response.statusText);                    
        }
      } catch (error) {
        console.log(error);
      }
    },
    async getRides() {
      // Call API to get the max 10 last completed rides
      try {
        let response = await fetch(process.env.VUE_APP_API_SERVER + '/booking/v5/rides?status=Completed', {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'apiKey': process.env.VUE_APP_API_KEY
          }          
        })

        if(response.ok) {
          return response.json()
        } else {
          console.log('Server returned ' + response.status + ' : ' + response.statusText);                    
        }
      } catch (error) {
        console.log(error);
      }
    },
    async getAnnouncements() {
      // Call API to get the active traveler announcements
      try {
        let response = await fetch(process.env.VUE_APP_API_SERVER + '/transportation/v1/announcements', {
          method: 'GET',
          headers: {
            'Content-Type': 'application/json',
            'apiKey': process.env.VUE_APP_API_KEY
          }          
        })

        if(response.ok) {
          return response.json()
        } else {
          console.log('Server returned ' + response.status + ' : ' + response.statusText);                    
        }
      } catch (error) {
        console.log(error);
      }
    },
    getIcon(format) {
      return divIcon({
        className: "custom-marker",
        iconAnchor: [20, 60],
        html: `<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 20 34.892337" height="60" width="40">
      <g transform="translate(-814.59595,-274.38623)">
        <g transform="matrix(1.1855854,0,0,1.1855854,-151.17715,-57.3976)">
          <path d="m 817.11249,282.97118 c -1.25816,1.34277 -2.04623,3.29881 -2.01563,5.13867 0.0639,3.84476 1.79693,5.3002 4.56836,10.59179 0.99832,2.32851 2.04027,4.79237 3.03125,8.87305 0.13772,0.60193 0.27203,1.16104 0.33416,1.20948 0.0621,0.0485 0.19644,-0.51262 0.33416,-1.11455 0.99098,-4.08068 2.03293,-6.54258 3.03125,-8.87109 2.77143,-5.29159 4.50444,-6.74704 4.56836,-10.5918 0.0306,-1.83986 -0.75942,-3.79785 -2.01758,-5.14062 -1.43724,-1.53389 -3.60504,-2.66908 -5.91619,-2.71655 -2.31115,-0.0475 -4.4809,1.08773 -5.91814,2.62162 z" style="fill:${format.color};stroke:${format.strokeColor};"/>
          <circle r="3.0355" cy="288.25278" cx="823.03064" id="path3049" style="display:inline;fill:${format.circleColor};"/>          
        </g>
      </g>
    </svg>`
      });
    },
    close() {
      this.reset()
    }
  }
}
</script>

<style>
.badge-shuttle {
  z-index: 200;
  margin-top: -20px;
  margin-left: 20px;
}
.custom-marker{
   background-color: transparent;
}
.leaflet-tooltip {
  padding: 2px !important;
}
.announcements-list {
  max-height: calc(100vh - 743px);
  overflow: scroll;
}
</style>