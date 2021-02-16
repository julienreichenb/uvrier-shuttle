<template>
    <b-modal id="book-modal" size="xl" centered header-bg-variant="warning" hide-footer @hide="hide">
        <template #modal-title>
            <h1 class="display-4 font-weight-bold">
                <font-awesome-layers class="fa-lg mr-3 align-bottom">
                    <font-awesome-icon icon="circle" class="text-success" />
                    <font-awesome-icon icon="circle" transform="shrink-2" color="white" />
                    <font-awesome-icon icon="bookmark" class="text-success" transform="shrink-8"  />
                </font-awesome-layers>
                <span>Réserver un trajet</span>
            </h1>
        </template>
        <b-form>
            <b-row class="my-5">
                <b-col>
                    <b-form-group label="Départ" label-for="start">
                        <b-input-group class="mt-3" size="lg">
                            <template #prepend>
                                <b-input-group-text class="bg-warning">
                                    <font-awesome-icon icon="map-marker-alt" class="large-text" />
                                </b-input-group-text>
                            </template>
                            <span v-if="origin" class="large-text pt-1 ml-3 font-weight-bold">{{ origin.name }}</span>
                        </b-input-group>
                    </b-form-group>
                </b-col>
                <b-col>
                    <b-form-group label="Destination" label-for="destination">
                        <b-input-group class="mt-3" size="lg">
                            <template #prepend>
                                <b-input-group-text class="bg-warning">
                                    <font-awesome-icon icon="flag" class="large-text" />
                                </b-input-group-text>
                            </template>
                            <b-form-select id="destination" v-model="selectedDestinationId" required :options="destinationOptions" class="large-text" @change="computeDeparture()" />
                        </b-input-group>
                    </b-form-group>
                </b-col>
            </b-row>
            <b-row class="my-5">
                <b-col>
                    <b-form-group label="Nombre de passagers" label-for="passengers">
                        <b-input-group class="mt-3" size="lg">
                            <template #prepend>
                                <b-input-group-text class="bg-warning">
                                    <font-awesome-icon icon="user" class="large-text" />
                                </b-input-group-text>
                            </template>
                            <b-form-select id="passengers" v-model="selectedNumber" required :options="numberOfPassengers" class="large-text" @change="computeDeparture()" />
                        </b-input-group>
                    </b-form-group>
                </b-col>
            </b-row>
            <b-row class="my-5" v-if="selectedDestinationId && selectedNumber">
                <b-col sm="12" class="mb-5">
                    <b-form-group label="Prochain départ" label-for="time-departure">
                        <b-input-group class="mt-3" size="lg">
                            <template #prepend>
                                <b-input-group-text class="bg-warning">
                                    <font-awesome-icon icon="clock" class="large-text" />
                                </b-input-group-text>
                            </template>                            
                            <span v-if="!loadingDepartureTimes && departureTimes" class="large-text pt-1 ml-3 font-weight-bold">Départ prévu entre {{ departureTimes[0] }} et {{ departureTimes[1] }}</span>
                            <span v-if="!loadingDepartureTimes && arrivalTimes" class="large-text pt-1">(arrivée à destination prévue entre {{ arrivalTimes[0] }} et {{ arrivalTimes[1] }}).</span>
                            <span v-if="!loadingDepartureTimes && !departureTimes && !arrivalTimes" class="large-text pt-1">Aucun trajet disponible.</span>                        
                            <span v-if="loadingDepartureTimes" class="large-text pt-1 ml-3">{{ computingDepartureTimes }}</span>
                        </b-input-group>
                    </b-form-group>
                </b-col>
                <b-col sm="8">
                    <b-button class="larger-text p-4" block variant="success" size="lg" :disabled="!departureTimes || !arrivalTimes || !quotes || loadingBookingStatus" @click="book()">
                        <font-awesome-icon icon="check" color="white" class="mr-3" />
                        <span>Confirmer la réservation</span>
                    </b-button>
                </b-col>
                <b-col sm="4">
                    <b-button class="larger-text p-4" block variant="secondary" size="lg" @click="reset()">
                        <font-awesome-icon icon="times" color="white" class="mr-3" />
                        <span>Annuler</span>
                    </b-button>                
                </b-col>
            </b-row>
        </b-form>
    </b-modal>
</template>

<script>
import moment from 'moment'
import { v4 as uuidv4 } from 'uuid'
export default {
    props: {
        show: { type: Boolean, default: false },
    },
    data() {
        return {
            quotes: null,
            stops: null,
            origin: null,            
            selectedDestinationId: null,
            selectedNumber: null,
            departureTimes: null,
            arrivalTimes: null,
            loadingDepartureTimes: false,
            loadingBookingStatus: false,
            computingDepartureTimes: 'Calcul de l\'horaire...',
            destinationOptions: [
                { text: 'Choisissez une destination', value: null },
            ],
            numberOfPassengers: [
                { text: 'Choisissez un nombre de passagers', value: null },
                { text: '1 personne', value: 1 },
                { text: '2 personnes', value: 2 },
                { text: '3 personnes', value: 3 },
                { text: '4 personnes', value: 4 },
            ]
        }
    },
    async created() {
        await this.loadAvailableDestinations()        
    },
    methods: {
        async computeDeparture() {
            this.departureTimes = null
            this.arrivalTimes = null
            this.quotes = null

            if (!this.selectedDestinationId || !this.selectedNumber) return
            
            this.loadingDepartureTimes = true
            // Call API to know when is the next available shuttle, considering destination/passenger number            
            let quotes = await this.createQuote()                
            if(quotes.length > 0) {
                this.quotes = quotes
                this.departureTimes = [moment(this.quotes[0].journeyEstimate.startTime.earliest).format('HH:mm'), moment(this.quotes[0].journeyEstimate.startTime.latest).format('HH:mm')]
                this.arrivalTimes = [moment(this.quotes[0].journeyEstimate.finishTime.earliest).format('HH:mm'), moment(this.quotes[0].journeyEstimate.finishTime.latest).format('HH:mm')]
            }
            this.loadingDepartureTimes = false
        },
        async loadAvailableDestinations() {
            // Call API to get the available destinations
            let stops = await this.getStops()
            if(stops) {
                this.stops = stops
                    this.origin = this.stops.find(s => s.name.toLowerCase().includes(process.env.VUE_APP_ORIGIN_FIND_NAME))
                    this.destinationOptions = this.destinationOptions.concat(this.stops.filter(s => s.id !== this.origin.id)
                        .map(s => {
                            return {
                                value: s.id,
                                text: s.name
                            }
                        })
                        .sort((a,b) => a.text.localeCompare(b.text)))
            } else {
                this.origin = null
                this.destinations = null
            }
        },
        reset() {
            this.selectedNumber = null
            this.selectedDestinationId = null
            this.departureTimes = null
            this.arrivalTimes = null
            this.quotes = null            
        },
        async book() {
            const destination = this.stops.find(s => s.id === this.selectedDestinationId)
            if(!this.selectedDestinationId || !this.selectedNumber || !this.quotes || this.quotes.length < 1 || !destination) return
            
            this.loadingBookingStatus = true
            //Call API to create a booking            
            let newBooking = await this.createBooking(this.quotes[0].id)
            
            if(newBooking) {                
                let booking
                let ride
                let confirmation
                let status

                do {
                    booking = await this.getBooking(newBooking.bookingID)
                    status = booking.status.toLowerCase()
                    
                    if(status === 'accepted') {
                        if(booking.journeyActivities) {
                            ride = booking.journeyActivities.find(j => j.type.toLowerCase() === 'ride')
                        }                        
                    }

                    await this.sleep(2000)
                } while (status === 'created' || (status === 'accepted' && (!ride || ride.status.toLowerCase() !== 'scheduled')))
                
                if(booking.status.toLowerCase() === 'accepted') {
                    confirmation = { id: booking.id, success: true, destination: ride.destination.stopName, departureTimes: [moment(ride.plannedPickupTime.earliest).format('HH:mm'), moment(ride.plannedPickupTime.latest).format('HH:mm')], vehicle: ride.vehicleDetails.name }
                } else {
                    confirmation = { success: false, destination: destination.name }
                }
                
                this.$emit('booking', confirmation)
                this.reset()                
            }            
            this.loadingBookingStatus = false
        },        
        async createQuote() {
            // Call API to create a quote
            var destination = this.stops.find(s => s.id === this.selectedDestinationId)
            
            try {
                let response = await fetch(process.env.VUE_APP_API_SERVER + '/booking/v5/quotes', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'apiKey': process.env.VUE_APP_API_KEY
                    },
                    body: JSON.stringify({
                        'origin': {
                            'location': {
                                'lat': this.origin.location.lat,
                                'lon': this.origin.location.lon
                            },
                            'label': this.origin.name
                        },
                        'destination': {
                            'location': {
                                'lat': destination.location.lat,
                                'lon': destination.location.lon
                            },
                            'label': destination.name
                        },
                        'numberOfTravelers': this.selectedNumber
                    })
                })

                if(response.ok) {
                    return response.json()
                } else {
                    console.log('Server returned ' + response.status + ' : ' + response.statusText);                    
                }
            } catch (error) {
                console.log(error)
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
                console.log(error)
            }
        },
        async createBooking(quoteId) {
            //Call API to create a booking
            try {
                let response = await fetch(process.env.VUE_APP_API_SERVER + '/booking/v5/bookings', {
                    method: 'POST',
                    headers: {
                        'Content-Type': 'application/json',
                        'apiKey': process.env.VUE_APP_API_KEY
                    },
                    body: JSON.stringify({
                        'quoteID': quoteId,
                        'traveler': {
                            'id': 'display-' + uuidv4()
                        },
                        'source': 'API'
                    })
                })

                if(response.ok) {
                    return response.json()
                } else {
                    console.log('Server returned ' + response.status + ' : ' + response.statusText);                    
                }
            } catch (error) {
                console.log(error)
            }
        },
        async getBooking(bookingId) {
            // Call API to get a booking
            try {
                let response = await fetch(process.env.VUE_APP_API_SERVER + '/booking/v5/bookings/' + bookingId, {
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
                console.log(error)
            }            
        },
        sleep(ms) {
            return new Promise(resolve => setTimeout(resolve, ms))
        },
        hide() {
            this.reset()
        }
    }
}
</script>

<style>
.large-text {
    font-size: 1.6em !important;
}
label, .larger-text {
    font-size: 2.2em !important;
}
</style>