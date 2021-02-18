<template>
    <b-modal id="info-modal" size="xl" centered ok-only header-bg-variant="warning">
        <template #modal-title>
            <h1>
                <font-awesome-layers class="fa-lg mr-3 align-bottom">
                    <font-awesome-icon icon="circle" class="text-primary" />
                    <font-awesome-icon icon="circle" transform="shrink-2" color="white" />
                    <font-awesome-icon icon="info" class="text-primary" transform="shrink-6"  />
                </font-awesome-layers>
                <span>Informations sur la navette autonome</span>
            </h1>
        </template>
        <template #modal-footer="{ ok }">
            <b-button class="large-text" size="lg" variant="secondary" @click="ok()">
                <font-awesome-icon icon="times" color="white" class="mr-3" />
                <span>Fermer</span>
            </b-button>
        </template>
        <h2>Le projet</h2>
        <p class="lead"><strong>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus efficitur fringilla libero, id iaculis nunc feugiat vel. Nunc vitae dignissim sem. Fusce tempor bibendum ipsum nec placerat. Nullam tempor suscipit leo. Aliquam id suscipit risus. Nullam ac ex in odio elementum pharetra. Cras at ex vehicula, viverra orci sit amet, auctor dolor. Cras fringilla quam eleifend elit finibus porta.</strong></p>
        <h2>Comment l'utiliser ?</h2>
        <p class="lead"><strong>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus efficitur fringilla libero, id iaculis nunc feugiat vel. Nunc vitae dignissim sem. Fusce tempor bibendum ipsum nec placerat. Nullam tempor suscipit leo. Aliquam id suscipit risus. Nullam ac ex in odio elementum pharetra. Cras at ex vehicula, viverra orci sit amet, auctor dolor. Cras fringilla quam eleifend elit finibus porta.</strong></p>
        <h2>Partenaires</h2>
        <p class="lead"><strong>Lorem ipsum dolor sit amet, consectetur adipiscing elit. Vivamus efficitur fringilla libero, id iaculis nunc feugiat vel. Nunc vitae dignissim sem. Fusce tempor bibendum ipsum nec placerat. Nullam tempor suscipit leo. Aliquam id suscipit risus. Nullam ac ex in odio elementum pharetra. Cras at ex vehicula, viverra orci sit amet, auctor dolor. Cras fringilla quam eleifend elit finibus porta.</strong></p>
    </b-modal>
</template>

<script>
export default {

}
</script>

<style>
</style>