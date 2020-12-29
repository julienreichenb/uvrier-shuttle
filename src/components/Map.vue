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
      <l-marker :lat-lng="youAreHere" />
      <l-marker 
        v-for="shuttle in shuttlePositions" 
        :key="shuttle.id" 
        :lat-lng="shuttle.position">
        <l-icon>
          <font-awesome-layers>
            <font-awesome-icon icon="circle" class="fa-3x" />
            <font-awesome-icon icon="circle" class="fa-3x text-white" transform="shrink-1" />
            <font-awesome-icon icon="bus" class="fa-3x" :class="shuttle.busy ? 'text-danger' : 'text-success'" transform="shrink-6" />
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
import { latLng } from "leaflet"
import InfoModal from './InfoModal'
import BookModal from './BookModal'
import ConfirmModal from './ConfirmModal'
export default {
  components: {
  InfoModal,
  BookModal,
  ConfirmModal,
  },
  mounted() {
    // Initial load
    this.loadShuttlesPositions()
    // Reoad Shuttles' position every 15 seconds
    setInterval(() => {
      this.loadShuttlesPositions()
    }, 15000);
  },
  data() {
    return {
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
      youAreHere: latLng(46.2515022, 7.4194519),
      shuttlePositions: null,
      showBook: false,
      showConfirm: false,
      booked: null,
      shuttleFree: 0,
      shuttleBusy: 0,
      avgWaitingTime: 0,
      liveInfo: 'Traffic régulier.'
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
      setTimeout(() => {
          // Simulating API delay
          const apiResult = [
            { id: 1, busy: true, position: latLng(46.251236, 7.4299814) },
            { id: 2, busy: false, position: latLng(46.2505541, 7.4096985) },
            { id: 3, busy: false, position: latLng(46.2521233, 7.4094444) },
            { id: 4, busy: false, position: latLng(46.2522791, 7.4291195) },
            { id: 5, busy: true, position: latLng(46.2538857, 7.4199932) },
            { id: 6, busy: false, position: latLng(46.2544875, 7.42915647) },
            { id: 7, busy: true, position: latLng(46.2525656, 7.4214758) },
            { id: 8, busy: true, position: latLng(46.2505421, 7.4193632) },
          ]
          this.shuttlePositions = apiResult
          this.shuttleBusy = apiResult.filter((s) => s.busy === true).length
          this.shuttleFree = apiResult.length - this.shuttleBusy
          this.avgWaitingTime = 2
      }, 400)
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