<template>
    <div>
        <b-card no-body>
            <b-card-header class="bg-warning border-dark border-bottom">
                <h1 class="display-3">SmartShuttle Uvrier</h1>
                <h1 class="display-5">Réservez votre trajet gratuit ici !</h1>
            </b-card-header>
            <b-card-body class="p-0">
                <Map />
            </b-card-body>
        </b-card>
    </div>
</template>

<script>
import Map from './Map'
export default {
    components: {
        Map,
    }
}
</script>

<style>

</style>