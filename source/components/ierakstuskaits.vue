<template>
  <div id="postcntbox">
    <p>Kopējais ierakstu skaits datubāze:</p>
    <p id="postcount" @updateview="dosmth">{{ postcount }}</p>
  </div>
</template>

<script setup>
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
      //this.$emit(`pskaits`, this.postcount);
      fetch(`http://localhost:3000/api/postscount/`, {
        method: 'GET',
        headers: {
          'Content-Type': 'application/json'
        }
      })
        .then((response) => response.json())
        .then((data) => {
          this.postcount = data.posts[0].postscount
        })

        .catch((error) => {
          console.error('Error:', error)
        })
    },
    getpost() {
      this.getpostcount()
    }
  }
}
</script>
