<template>
  <div id="Pubs">
    <div v-if="loading" class="d-flex justify-content-center mb-3">
      <b-spinner variant="primary"></b-spinner>
    </div>
    <div v-else class="container-fluid pub-content">
      <b-row align-h="between">
        <b-col cols="6">
          <h3>Journal Articles ({{ works.length }})</h3>
        </b-col>
        <b-col cols="6" class="text-right">
          <button @click="toggleAuthors">{{ showAuthors ? 'Hide Authors' : 'Show Authors' }}</button> <!-- Single Toggle button for all -->
        </b-col>
        </b-row>
        <b-list-group>
        <b-list-group-item v-for="(work, index) in works" :key="index">
          <PubItem
            :pub="work['work-summary'][0]"
            :doi="work['work-summary'][0]['external-ids']['external-id'][0]['external-id-value']"
            :show-authors="showAuthors"> <!-- Pass showAuthors prop -->
          </PubItem>
        </b-list-group-item>
      </b-list-group>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import BootstrapVue from 'bootstrap-vue';
import PubItem from './PubItem'
Vue.use(BootstrapVue);

export default {
  name: 'Bloom',
  props: {},
  components: {
    PubItem,
  },
  data() {
    return {
      works: [],
      loading: true,
      showAuthors: false, // New data property to control visibility of authors
    };
  },
  mounted() {
    this.getOrcid('0000-0001-7283-271X');  // Replace with your own ORCID !
  },
  methods: {
    toggleAuthors() { // Method to toggle author visibility
      this.showAuthors = !this.showAuthors;
    },
    getOrcid(orcid) {
      const options = {
          method: 'GET',
          headers: new Headers({'accept': 'application/json'})
      };
      fetch(`https://pub.orcid.org/v3.0/${orcid}/works`, options)
      .then((resp) => resp.json())
      .then(data => {
        this.works = data.group;
        this.loading = false;
      });
    }
  },
};
</script>

<style>
/* ===================================
publications
==================================== */
.pub-content {
    background:transparent;
    position:relative;
    margin-left:10px;
    margin-bottom: 10px;
    padding:40px 60px 0px 0px;
}

</style>
