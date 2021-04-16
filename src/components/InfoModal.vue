<template>
    <b-modal id="info-modal" size="xl" centered ok-only header-bg-variant="warning">
        <template #modal-title>
            <h1>
                <font-awesome-layers class="fa-lg mr-3 align-bottom">
                    <font-awesome-icon icon="circle" class="text-primary" />
                    <font-awesome-icon icon="circle" transform="shrink-2" color="white" />
                    <font-awesome-icon icon="info" class="text-primary" transform="shrink-6"  />
                </font-awesome-layers>
                <span>Informations sur les Smartshuttle</span>
            </h1>
        </template>
        <template #modal-footer="{ ok }">
            <b-button class="large-text" size="lg" variant="secondary" @click="ok()">
                <font-awesome-icon icon="times" color="white" class="mr-3" />
                <span>Fermer</span>
            </b-button>
        </template>
        <h2>Les Smartshuttle à la demande débarquent à Uvrier. Vivez l’innovation!</h2>
        <p class="lead"><strong>Depuis avril, les navettes électriques autonomes circulent à la demande à Uvrier. Dans le cadre d’une exploitation test se déroulant sur plusieurs mois, les habitants et les touristes peuvent ainsi réserver gratuitement ces navettes automatisées pour le parcours de leur choix.</strong></p>
        <p class="lead"><strong>Ce test fait partie d’un projet conduit au niveau européen en collaboration avec la Ville de Sion, CarPostal, Mobility Lab et la HES-SO.</strong></p>
        <p class="lead"><strong>Les deux navettes fonctionnent du lundi au vendredi de 7h00 à 10h00 et de 13h00 à 18h00.</strong></p>
        <p class="lead"><strong>Informations et service clients: +41 76 804 80 72</strong></p>
        <p class="lead"><i>Cette application a été conçue par l’Institut de recherche en informatique de gestion de la HES-SO Valais-Wallis qui est spécialisé dans le développement de systèmes d’information.</i></p>        
    </b-modal>
</template>

<script>
export default {

}
</script>

<style>
</style>