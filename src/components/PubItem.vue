<template>
  <b-list-group v-if="pub">
    <b-row align-h="between">
      <b-col cols="10">
        <div class="d-flex align-items-center">
          <!-- Vertical Toggle Switch -->
          <label class="switch">
            <input type="checkbox" v-model="showAuthors" @change="toggleAuthors">
            <span class="slider"></span>
          </label>
          <a class="mb-1 ml-2" :href="pub['url']['value']" target="_blank">
            {{ pub['title']['title']['value'] }}
          </a>
        </div>
        <div v-if="showAuthors"> <!-- Conditional rendering based on showAuthors -->
          <template v-for="(author, index) in authorList">
            <span v-bind:key="index" :style="styleAuthor(author)">{{ author['given'].charAt(0) }} {{ author['family'] }}</span><!--
            --><span>{{ index == authorList.length - 1 ? '.': ', ' }}</span>
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

export default {
  name: 'Bloom',
  props: {doi: {type: String},
          pub: {type: Object}},
  data() {
    return {
      works: [],
      loading: true,
      authorList: [],
      showAuthors: false, // New data property to control visibility
    };
  },
  mounted() {
    this.getCrossref(this.doi);
    // Check DOI properly passed to child component
    // console.log(this.doi);
    let dimensionScript = document.createElement('script')
    dimensionScript.setAttribute('src', 'https://badge.dimensions.ai/badge.js')
    document.head.appendChild(dimensionScript)
  },
  methods: {
    toggleAuthors() { // New method to toggle author visibility
      this.showAuthors = !this.showAuthors;
    },
    getCrossref(doi) {
      const options = {
          method: 'GET',
          headers: new Headers({'accept': 'application/json'})
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
      var style = {}; // Replace with your own family name !
      if (author['family'] == 'Karakuzu') {
        style.color = 'black',
        style.textDecoration = 'underline'
      }
      return style;
    }
  },
};
</script>

<style>
.switch {
  position: relative;
  display: inline-block;
  width: 34px;
  height: 60px; /* Adjust height for vertical switch */
  margin-right: 10px; /* Space between switch and title */
}

.switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background-color: #ccc;
  transition: .4s;
  border-radius: 34px; /* Rounded edges */
}

.slider:before {
  position: absolute;
  content: "";
  height: 26px; /* Adjust height for vertical switch */
  width: 26px; /* Adjust width for vertical switch */
  left: 4px;
  bottom: 4px;
  background-color: white;
  transition: .4s;
  border-radius: 50%; /* Circular knob */
}

input:checked + .slider {
  background-color: #2196F3; /* Color when checked */
}

input:checked + .slider:before {
  transform: translateY(26px); /* Move knob down when checked */
}
</style>
