<template>
    <b-modal v-model="show" size="xl" centered header-bg-variant="warning" hide-footer @hide="hide">
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
                            <b-form-select id="passengers" v-model="selectedNumber" required :options="numberOptions" class="large-text" @change="computeDeparture()" />
                        </b-input-group>
                    </b-form-group>
                </b-col>
            </b-row>
            <b-row class="my-5" v-if="selectedDestinationId && selectedNumber">
                <b-col sm="12" class="mb-5">
                    <b-form-group label="Prochain départ dans" label-for="time-departure">
                        <b-input-group class="mt-3" size="lg">
                            <template #prepend>
                                <b-input-group-text class="bg-warning">
                                    <font-awesome-icon icon="clock" class="large-text" />
                                </b-input-group-text>
                            </template>
                            <span v-if="!loadingTimeDeparture" class="large-text pt-1 ml-3 font-weight-bold">{{ timeDeparture }}</span>
                            <span v-else class="large-text pt-1 ml-3">{{ computingTimeDeparture }}</span>
                        </b-input-group>
                    </b-form-group>
                </b-col>
                <b-col sm="8">
                    <b-button class="larger-text p-4" block variant="success" size="lg" :disabled="!timeDeparture" @click="book()">
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
import { API_KEY, API_SERVICE_ID, API_SERVER, ORIGIN_FIND_NAME } from '../constants'
export default {
    props: {
        show: { type: Boolean, default: false },
    },
    data() {
        return {
            stops: null,
            origin: null,
            selectedDestinationId: null,
            selectedNumber: null,
            timeDeparture: null,
            loadingTimeDeparture: false,
            computingTimeDeparture: 'Calcul du temps...',
            destinationOptions: [
                { text: 'Choisissez une destination', value: null },
            ],
            numberOptions: [
                { text: 'Choisissez un nombre de passagers', value: null },
                { text: '1 personne', value: 1 },
                { text: '2 personnes', value: 2 },
                { text: '3 personnes', value: 3 },
                { text: '4 personnes', value: 4 },
                { text: '5 personnes', value: 5 },
            ],
        }
    },
    created() {
        this.loadAvailableDestinations()
    },
    methods: {
        computeDeparture() {
            this.timeDeparture = null
            if (!this.selectedDestinationId || !this.selectedNumber) return
            // Call API to know when is the next available shuttle, considering destination/passenger number
            this.loadingTimeDeparture = true
            setTimeout(() => {
                // Simulating API delay
                const noTravelFound = false
                if(!noTravelFound) {
                    this.timeDeparture = 4 + ' minutes'
                } else {
                    this.timeDeparture = 'Aucun trajet disponible.'
                }
                this.loadingTimeDeparture = false
            }, 2000)
        },
        loadAvailableDestinations() {
            // Call API to get the available destinations
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
                    this.destinationOptions = [{ text: 'Choisissez une destination', value: null }]
                    alert('Server returned ' + response.status + ' : ' + response.statusText);
                }                
            })
            .then(response => {
                if(response){
                    this.stops = response
                    this.origin = this.stops.find(s => s.name.toLowerCase().includes(ORIGIN_FIND_NAME))                    
                    this.destinationOptions = this.destinationOptions.concat(this.stops.filter(s => s.id !== this.origin.id)
                        .map(s => {
                            return {
                                value: s.id,
                                text: s.name
                            }
                        })
                        .sort((a,b) => a.value.localeCompare(b.value)))
                }
            })
            .catch(err => {
                console.log(err);
            });                  
        },
        reset() {
            this.selectedNumber = null
            this.selectedDestinationId = null
            this.timeDeparture = null
        },
        book() {
            const destination = this.stops.find(s => s.id === this.selectedDestinationId)

            // Call API to book the trip
            setTimeout(() => {
                // Simulating API delay
                const apiAnswer = { id: 4312097, success: true, destination: destination.name, persons: this.selectedNumber, time: this.timeDeparture }
                this.$emit('booking', apiAnswer)
                this.reset()
            }, 4000)
        },
        hide() {
            this.reset()
            this.$emit('hide')
        },
    },
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