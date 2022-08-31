<template>
  <div>
    <a-button type="default" @click="getLocation">Get current position</a-button>
    <div v-if="curPosResponse" class="cur-pos-response">{{ curPosResponse }}</div>
    <form id="search-form" @submit="submitSearch">
      <input 
        id="search-text" 
        type="text" 
        placeholder="Search Locations"
        v-model="searchText"
      >
      <a-button type="primary" @click="submitSearch">Search</a-button>
    </form>
    
    <div class="delete-div" v-if="outputResults.length > 0" >
      <a-button danger :disabled="!hasSelected" @click="deleteHandler">
        Delete
      </a-button>
      <div v-if="hasSelected" class="num-of-selected">Selected {{ selectedRowKeys.length }} {{ (selectedRowKeys.length === 1 ? 'location' : 'locations') }}</div>
    </div>
    <a-table 
      v-if="outputResults.length > 0" 
      id="search-results-table"
      :columns="tableColumns"
      :data-source="outputResults"
      :row-selection="{ selectedRowKeys: selectedRowKeys, onChange: onSelectChange }"
    />
    <div v-else-if="outputResults.length <= 0 && searched" class="no-search-results">No results</div>

    <div v-if="outputResults.length > 0 && latestLocationInfo">{{ latestLocationInfo }}</div>

    <v-map :options="map">
      <!-- <v-marker
          :coordinates="coordinates"
          :options="{}"
          :popupOptions="{}"
      /> -->
    </v-map>
  </div>
</template>

<script>
import 'mapbox-gl/dist/mapbox-gl.css'
import 'v-mapbox/dist/v-mapbox.css'
// import mapbox from "mapbox-gl"
import { VMap } from "v-mapbox"

export default {
  name: 'App',
  components: {
    VMap
  },
  data () {
    return {
      searchText: '',
      curPosResponse: false,
      searchResponse: [],
      outputResults: [],
      tableColumns: [
        { title: 'Location', dataIndex: 'address', key: 'address' }
      ],
      selectedRowKeys: [],
      searched: false,
      baseUrl: 'https://api.mapbox.com/geocoding/v5/mapbox.places/',
      mapboxToken: 'pk.eyJ1Ijoia2V2aW44NTIiLCJhIjoiY2w3N3d3dHYzMDQ0YzNvcGx6b3Nndmg4NSJ9.3AuL3Enwv-Lt02i_8mDP4Q',
      mapStyle: 'mapbox://styles/mapbox/light-v10',
      map: {
        accessToken: 'pk.eyJ1Ijoia2V2aW44NTIiLCJhIjoiY2w3N3d3dHYzMDQ0YzNvcGx6b3Nndmg4NSJ9.3AuL3Enwv-Lt02i_8mDP4Q',
        style: 'mapbox://styles/mapbox/light-v10',
        center: [-79.40688, 43.73033],
        zoom: 1,
        maxZoom: 15,
        crossSourceCollisions: false,
        failIfMajorPerformanceCaveat: false,
        attributionControl: false,
        preserveDrawingBuffer: true,
        hash: false,
        minPitch: 0,
        maxPitch: 60,
        // projection: 'globe',
      },
      coordinates: [-79.40688, 43.73033],
      latestLocationInfo: ''
      // marker: {
      //   coordinates: [-79.40688, 43.73033],
      //   color:'blue'
      // }
    }
  },
  watch: {
    outputResults(newOutputResults) {
      if (newOutputResults.length> 0) {
        fetch(`https://api.timezonedb.com/v2.1/get-time-zone?key=W2M9HH219UX6&format=json&by=position&lat=${this.outputResults[0].coord[1]}&lng=${this.outputResults[0].coord[0]}`)
          .then(res => res.json())
          .then(timezoneData => {
            console.log(`Timezone of ${this.outputResults[0].address}: `, timezoneData.zoneName)
            this.latestLocationInfo = timezoneData.abbreviation + ', ' + timezoneData.formatted + ', ' + timezoneData.zoneName
          })
          .catch(err => alert(err))
      }
    }
  },
  computed: {
    hasSelected() {
      return this.selectedRowKeys.length > 0
    }
  },
  // created() {
  //   // We need to set mapbox-gl library here in order to use it in template
  //   this.mapbox = mapbox;
  // },
  methods: {
    submitSearch(event) {
      event.preventDefault()
      console.log("Search submitted: ", this.searchText)
      fetch(`${this.baseUrl}${this.searchText}.json?limit=10&proximity=ip&access_token=${this.mapboxToken}`)
        .then((response) => response.json())
        .then((data) => {
          console.log('Searched: ', data)
          // this.searchResponse = data.features
          this.outputResults = []
          // console.log("🚀 ~ file: App.vue ~ line 71 ~ .then ~ this.outputResults", this.outputResults)
          for (const location of data.features) {

            this.outputResults.push({
              key: location.id,
              address: location.place_name,
              coord: location.center
            })
          }
        })
        .catch((error) => console.warn(error.message))
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
      
    },
    onSelectChange (selectedRowKeys) {
      console.log('selectedRowKeys changed: ', selectedRowKeys)
      this.selectedRowKeys = selectedRowKeys
      console.log("🚀 ~ file: App.vue ~ line 121 ~ onSelectChange ~ this.selectedRowKeys", this.selectedRowKeys)
    },
    deleteHandler() {
      const removePos = []
      //record the index of the element to be removed in an array
      for (const x of this.selectedRowKeys) {
        removePos.push(this.outputResults.map((e) => { return e.key }).indexOf(x))
      }

      // remove selected locations in the outputResults array
      this.outputResults = this.outputResults.filter((value, index) => {
        return !removePos.includes(index)
      })
      
      // console.log('* outputResults: ', this.outputResults)

      //clean up (avoid highlighting selected location; avoid showing how many locations are selected)
      this.selectedRowKeys = []
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

.cur-pos-response, table, ul.ant-pagination, .delete-div {
  padding: 0 1rem;
}

#search-form, .cur-pos-response, #map, .no-search-results, #search-results-table, .delete-div {
  margin-top: 1rem;
}

#search-text {
  display: inline;
  width: 50vw;
  height: 32px;
  box-sizing: border-box;
  margin: 0 2rem 0 0;
  font-variant: tabular-nums;
  list-style: none;
  font-feature-settings: "tnum";
  position: relative;
  display: inline-block;
  min-width: 0;
  padding: 4px 11px;
  color: #000000d9;
  font-size: 14px;
  line-height: 1.5715;
  background-color: #fff;
  background-image: none;
  border: 1px solid #d9d9d9;
  border-radius: 2px;
  transition: all .3s;
}

#search-text:focus {
  border-color: #40a9ff;
  box-shadow: 0 0 0 2px rgba(24, 144, 255, .2);
  border-right-width: 1px!important;
  outline: 0;
}

.delete-div {
  text-align: left;
}

table {
  margin-inline: auto;
}

.num-of-selected {
  display: inline;
  margin-left: 1rem;
}

</style>
