<template>
  <div id="postcntbox">
    <p>Kopējais ierakstu skaits datubāze:</p>
    <p id="postcount" @updateview="dosmth">{{ postcount }}</p>
  </div>
</template>

<script setup>
import axios from 'axios'

const emit = defineEmits(['pskaits'])
</script>

<script>
export default {
  name: 'ierakstuskaits',
  expose: ['getpost'],
  setup() {
    const postcount = ref(0)
    return { postcount }
  },
  data() {
    return {
      postcount: 0
    }
  },
  mounted() {
    this.getpostcount()
  },
  methods: {
    async getpostcount() {
      const response = await axios.get(`http://localhost:3000/api/postscount/`)
      this.postcount = response.data.posts[0].postscount
      //this.$emit(`pskaits`, this.postcount);
    },
    getpost() {
      this.getpostcount()
    }
  }
}
</script>
