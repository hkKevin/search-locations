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
    
    <!-- <table v-if="searchResponse.length > 0" id="search-results-table">
      <tr v-for="location in searchResponse" :key="location.id">
        <td>{{ location.place_name }}</td>
      </tr>
    </table> -->
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
      outputResults: [],
      tableColumns: [
        { title: 'Location', dataIndex: 'address', key: 'address' }
      ],
      selectedRowKeys: [],
      searched: false,
      baseUrl: 'https://api.mapbox.com/geocoding/v5/mapbox.places/',
      mapboxToken: 'pk.eyJ1Ijoia2V2aW44NTIiLCJhIjoiY2w3N3d3dHYzMDQ0YzNvcGx6b3Nndmg4NSJ9.3AuL3Enwv-Lt02i_8mDP4Q'
    }
  },
  computed: {
    hasSelected() {
      return this.selectedRowKeys.length > 0
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
          // this.searchResponse = data.features
          this.outputResults = []
          console.log("🚀 ~ file: App.vue ~ line 71 ~ .then ~ this.outputResults", this.outputResults)
          for (const location of data.features) {
            this.outputResults.push({
              key: location.id,
              address: location.place_name,
            })
          }
          console.log("this.outputResults", this.outputResults)
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
      
    },
    onSelectChange (selectedRowKeys) {
      console.log('selectedRowKeys changed: ', selectedRowKeys)
      this.selectedRowKeys = selectedRowKeys
      console.log("🚀 ~ file: App.vue ~ line 121 ~ onSelectChange ~ this.selectedRowKeys", this.selectedRowKeys)
    },
    deleteHandler() {
      console.log('^ Delete something...')
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
