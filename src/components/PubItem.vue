<template>
  <b-list-group v-if="pub">
    <b-row align-h="between">
      <b-col cols="10">
        <div class="d-flex align-items-center">
          <!-- PrimeVue ToggleSwitch for showing/hiding authors -->
          <ToggleSwitch v-model="showAuthors" onLabel="Show Authors" offLabel="Hide Authors" />
          <a class="mb-1 ml-2" :href="pub['url']['value']" target="_blank">
            {{ pub['title']['title']['value'] }}
          </a>
        </div>
        <div v-if="showAuthors"> <!-- Conditional rendering based on showAuthors -->
          <template v-for="(author, index) in authorList" :key="index">
            <span :style="styleAuthor(author)">{{ author['given'].charAt(0) }} {{ author['family'] }}</span><!--
            --><span>{{ index == authorList.length - 1 ? '.' : ', ' }}</span>
          </template>
        </div>
        <div>
          <p v-if="pub['journal-title']" class="mb-1">
            <small>{{ pub['journal-title']['value'] }} ({{ pub['publication-date']['year']['value'] }})</small>
          </p>
          <p v-else class="mb-1">
            <small>Preprint ({{ pub['publication-date']['year']['value'] }})</small>
          </p>
        </div>
      </b-col>
      <b-col cols="2" align-self="center">
        <div class="d-flex float-right">
          <span class="__dimensions_badge_embed__"
            :data-doi="doi"
            data-style="small_circle"
            data-hide-zero-citations="true">
          </span>
        </div>
      </b-col>
    </b-row>
  </b-list-group>
</template>

<script>
import ToggleSwitch from 'primevue/toggleswitch'; // Importing ToggleSwitch from PrimeVue

export default {
  name: 'Bloom',
  components: {
    ToggleSwitch, // Registering the ToggleSwitch component
  },
  props: {
    doi: { type: String },
    pub: { type: Object },
  },
  data() {
    return {
      works: [],
      loading: true,
      authorList: [],
      showAuthors: false, // Data property to control visibility
    };
  },
  mounted() {
    this.getCrossref(this.doi);
    let dimensionScript = document.createElement('script');
    dimensionScript.setAttribute('src', 'https://badge.dimensions.ai/badge.js');
    document.head.appendChild(dimensionScript);
  },
  methods: {
    getCrossref(doi) {
      const options = {
        method: 'GET',
        headers: new Headers({ 'accept': 'application/json' }),
      };
      fetch(`https://api.crossref.org/works/${doi}`, options)
        .then((resp) => {
          if (!resp.ok) {
            throw new Error('Network response was not ok');
          }
          return resp.json();
        })
        .then(data => {
          // Check if authors exist and are in the expected format
          if (data.message && Array.isArray(data.message.author)) {
            this.authorList = data.message.author;
          } else {
            console.warn('No authors found for this DOI:', doi);
            this.authorList = []; // Set to empty if no authors found
          }
        })
        .catch(error => {
          console.error('Error fetching data:', error);
          this.authorList = []; // Set to empty on error
        });
    },
    styleAuthor(author) {
      var style = {};
      if (author['family'] === 'Karakuzu') {
        style.color = 'black';
        style.textDecoration = 'underline';
      }
      return style;
    },
  },
};
</script>

<style>
/* Add any additional styles if needed */
</style>
