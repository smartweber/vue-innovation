<template>
  <div class="hello">
    <form id="search">
      Search <input name="query" v-model="searchQuery">
    </form>
    <innovation-grid
      :data="gridData"
      :columns="gridColumns"
      :filter-key="searchQuery">
    </innovation-grid>
  </div>
</template>

<script>
import InnovationGrid from '@/_components/innovation-grid'
export default {
  name: 'Dashboard',
  components: { InnovationGrid },
  data () {
    return {
      searchQuery: '',
      gridColumns: ['name', 'power'],
      gridData: [
        { id: 1, name: 'Chuck Norris', power: Infinity },
        { id: 2, name: 'Bruce Lee', power: 9000 },
        { id: 3, name: 'Jackie Chan', power: 7000 },
        { id: 4, name: 'Jet Li', power: 8000 }
      ]
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
