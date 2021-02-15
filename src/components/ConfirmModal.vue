<template>
  <b-modal v-if="confirmation" 
    v-model="show" 
    size="xl" 
    centered 
    :header-bg-variant="confirmation.success ? 'success' : 'danger'" 
    hide-footer
    hide-header-close
    no-close-on-esc 
    no-close-on-backdrop>
        <template #modal-title>
            <h2 class="font-weight-normal">
                <font-awesome-layers class="fa-lg mr-3 align-bottom">
                    <font-awesome-icon icon="circle" class="black" />                    
                    <font-awesome-icon icon="circle" transform="shrink-2" color="white" />
                    <font-awesome-icon v-if="confirmation.success" icon="check" class="text-success" transform="shrink-6"  />
                    <font-awesome-icon v-else icon="times" class="text-danger" transform="shrink-6"  />
                </font-awesome-layers>
                <span>Trajet pour <strong>{{ confirmation.destination}}</strong> <span v-if="confirmation.success">réservé</span><span v-else>rejeté</span> !</span>
            </h2>                    
        </template>
        <h2 v-if="confirmation.success" class="p-3"><span>Départ prévu entre {{ confirmation.departureTimes[0] }} et {{ confirmation.departureTimes[1] }}.</span></h2>
        <h2 v-else class="p-3"><span>Aucun véhicule n'est disponible pour accepter cette réservation.</span></h2>
        <h5 v-if="confirmation.success" class="p-3">Réservation: {{ confirmation.id }}</h5>
    </b-modal>
</template>

<script>
export default {
    props: {
        confirmation: { type: Object, default: null },
        show: { type: Boolean, default: false }
    },
}
</script>

<style>

</style>