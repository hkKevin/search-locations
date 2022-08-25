<template>
  <div>
    <button @click="getLocation">Get current position</button>
    <div v-if="curPosResponse" class="cur-pos-response">{{ curPosResponse }}</div>
    <form id="search-form" @submit="submitSearch">
      <input 
        id="search-text" 
        type="text" 
        placeholder="Search Locations"
        v-model="searchText"
      >
      <input type="submit" value="Search">
    </form>
    
    <table v-if="searchResponse.length > 0" id="search-results-table">
      <tr v-for="location in searchResponse" :key="location.id">
        <td>{{ location.place_name }}</td>
      </tr>
    </table>
    <div v-else-if="searchResponse.length <= 0 && searched" class="no-search-results">No results</div>

    <div id="map">
      <iframe 
        width='100%' 
        height='100%' 
        src="https://api.mapbox.com/styles/v1/kevin852/cl784awq0002x14mgw9450gq5/draft.html?title=false&access_token=pk.eyJ1Ijoia2V2aW44NTIiLCJhIjoiY2w3N3d3dHYzMDQ0YzNvcGx6b3Nndmg4NSJ9.3AuL3Enwv-Lt02i_8mDP4Q&zoomwheel=true#10/43.7171/-79.3828" 
        title="Basic" 
        style="border:none;">
      </iframe>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      searchText: '',
      curPosResponse: false,
      searchResponse: [],
      searched: false,
      baseUrl: 'https://api.mapbox.com/geocoding/v5/mapbox.places/',
      mapboxToken: 'pk.eyJ1Ijoia2V2aW44NTIiLCJhIjoiY2w3N3d3dHYzMDQ0YzNvcGx6b3Nndmg4NSJ9.3AuL3Enwv-Lt02i_8mDP4Q'
    }
  },
  methods: {
    submitSearch(event) {
      event.preventDefault()
      console.log("Search submitted: ", this.searchText)
      fetch(`${this.baseUrl}${this.searchText}.json?limit=10&proximity=ip&access_token=${this.mapboxToken}`)
        .then((response) => response.json())
        .then((data) => {
          console.log('Searched: ', data)
          this.searchResponse = data.features
        })
        .catch((error) => alert(error.message))
      this.searched = true
    },
    getLocation() {
      // alert('getLocation')
      if (navigator.geolocation) {
      // alert('has navigator.geolocation')
        navigator.geolocation.getCurrentPosition((position) => {
          console.log("🚀 ~ file: App.vue ~ line 35 ~ navigator.gerlocation.getCurrentPosition ~ position", position)
          const { latitude, longitude } = position.coords;
          fetch(`${this.baseUrl}${longitude},${latitude}.json?access_token=${this.mapboxToken}`)
            .then((response) => response.json())
            .then((data) => {
              console.log(data)
              const location = data.features[0]
              if (data.features.length > 0 && location.place_name) {
                this.curPosResponse = location.place_name
              }
            })
            .catch((error) => alert(error.message))
        }, (err) => {
          console.log(err.message)
          this.curPosResponse = err.message
        })
      } else {
        alert('Geolocation not supported.')
      }
      
    }
  }
}
</script>

<style>
body {
  margin: 0;
}

#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 30px;
}

#map {
  height: 80vh;
}

#search-form, .cur-pos-response, #map, .no-search-results, #search-results-table {
  margin-top: 1rem;
}

table {
  margin-inline: auto;
}

tr {
  text-align: left;
  line-height: 1.5rem;
}

.cur-pos-response, table {
  padding: 0 1rem;
}
</style>
