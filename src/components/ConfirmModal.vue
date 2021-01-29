<template>
  <b-modal v-if="booking" 
    v-model="show" 
    size="xl" 
    centered 
    :header-bg-variant="booking.success ? 'success' : 'danger'" 
    hide-footer 
    no-close-on-backdrop>
        <template #modal-title>
            <h2 class="font-weight-normal">
                <font-awesome-layers class="fa-lg mr-3 align-bottom">
                    <font-awesome-icon icon="circle" class="text-primary" />
                    <font-awesome-icon icon="circle" transform="shrink-2" color="white" />
                    <font-awesome-icon icon="check" class="text-primary" transform="shrink-6"  />
                </font-awesome-layers>
                <span>Trajet pour <strong>{{ booking.destination}}</strong> réservé !</span>
            </h2>                    
        </template>
        
        <h2 class="p-3">Horaire estimé: {{ booking.time }}.</h2>
        <h5 class="p-3">Réservation: {{ booking.id }}</h5>
    </b-modal>
</template>

<script>
export default {
    props: {
        booking: { type: Object, default: null },
        show: { type: Boolean, default: false },
    },
}
</script>

<style>

</style>