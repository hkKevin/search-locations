<template>
  <div>
    <form @submit="submitSearch">
      <input 
        id="search-text" 
        type="text" 
        placeholder="Search Locations"
        v-model="searchText"
      >
      <input type="submit" value="Search">
    </form>
    <p>Searching: {{ searchText }}</p>
  </div>
</template>

<script>
export default {
  name: 'App',
  data () {
    return {
      searchText: ''
    }
  },
  methods: {
    submitSearch(event) {
      event.preventDefault()
      console.log("Search submitted: ", this.searchText)
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
