<template>
  <div>
    <input id="searchbar" type="text" placeholder="Search" />
    <button id="search-btn" @click="dosearch">Meklēt</button>
  </div>
</template>

<script>
export default {
  name: 'search',
  components: {},
  emits: ['search'],
  setup() {
    return {}
  },
  methods: {
    dosearch() {
      this.$emit('search')
    }
  }
}
</script>
