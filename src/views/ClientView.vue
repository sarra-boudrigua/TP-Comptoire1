<template>
    <main>
        <div>
            <h1>Liste des Clients - Page {{ currentPage }} / {{ totalPages }}</h1>
            <table>
                <caption>Liste des clients</caption>
                <tr>
                    <th>Code</th>
                    <th>Société</th>
                    <th>Contact</th>
                    <th>Ville</th>
                </tr>
                <!-- Si le tableau des clients est vide -->
                <tr v-if="slicedClients.length === 0">
                    <td colspan="4">Veuillez patienter, chargement des clients...</td>
                </tr>
                <!-- Si le tableau des clients n'est pas vide -->
                <tr v-for="client in slicedClients" :key="client.code">
                    <td>{{ client.code }}</td>
                    <td>{{ client.societe }}</td>
                    <td>{{ client.contact }}</td>
                    <td>{{ client.adresse.ville }}</td>
                </tr>
            </table>
            <div class="pagination">
                <button @click="goToPage(1)" :disabled="currentPage === 1">Début</button>
                <button @click="goToPage(currentPage - 1)" :disabled="currentPage === 1">Précédent</button>
                <button @click="goToPage(currentPage + 1)" :disabled="currentPage === totalPages">Suivant</button>
                <button @click="goToPage(totalPages)" :disabled="currentPage === totalPages">Fin</button>
            </div>
        </div>
    </main>
</template>

<script setup>
import { reactive, onMounted, ref, watch } from "vue";
import { doAjaxRequest } from "@/api";

let data = reactive({
    listeClients: []
});

const itemsPerPage = 5;
const currentPage = ref(1);
const totalPages = ref(1);

function showError(error) {
    console.log("Erreur : status %d", error.status);
    console.log(error.body);
    alert(error.message);
}

function chargeClients() {
    doAjaxRequest("/api/clients")
        .then((json) => {
            data.listeClients = json._embedded.clients;
            updateTotalPages();
            goToPage(2);
        })
        .catch(showError);
}

function updateTotalPages() {
    totalPages.value = Math.ceil(data.listeClients.length / itemsPerPage);
}

function goToPage(pageNumber) {
    currentPage.value = pageNumber;
}

const slicedClients = ref([]);
watch([currentPage, data.listeClients], () => {
    const startIndex = (currentPage.value - 1) * itemsPerPage;
    const endIndex = startIndex + itemsPerPage;
    slicedClients.value = data.listeClients.slice(startIndex, endIndex);
});


onMounted(chargeClients);

</script>

<style scoped>
td,
th {
    border: 1px solid #ddd;
    padding: 8px;
}

th {
    padding-top: 12px;
    padding-bottom: 12px;
    text-align: left;
    background-color: #232623;
    color: rgb(255, 255, 255);
}

.pagination {
    margin-top: 10px;
}

.pagination button {
    margin-right: 5px;
}
</style>