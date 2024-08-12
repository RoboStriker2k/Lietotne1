<template>
  <div class="container">
    <div class="header">
      <header>
        <h1>Lietotne 1 -VUE</h1>
      </header>
    </div>
    <div id="nav">
      <div id="nav-bar">
        <button v-if="!showupload" @click="showuploadfn">Pievienot</button>
        <button v-if="showupload" @click="showuploadfn">Aizvert pievienot</button>
        <button @click="Updateview">Atsvaidzinat</button>
      </div>
      <div>
        <input id="searchbar" type="text" placeholder="Search" />
        <button id="search-btn" @click="dosearch">Meklēt</button>
      </div>
    </div>
    <div>
      <div v-if="showupload">
        <upl @closeupload="showuploadfn" @update="Updateview" />
      </div>
    </div>
    <div id="content">
      <ierakstuskaits @pskaits="(pskaits) => (ierskaits = pskaits)" ref="ierskaits" />
      <p>Lietotnes ieraksti</p>
      <div id="dyncontent">
        <ieraksts @update="Updateview" @editfn="editfn" ref="ier" />
        <editcomp :Posttatus="editpostdata" ref="edit" @update="Updateview" />
      </div>
    </div>
  </div>

  <footer>
    <p>Lietotni veidoja Roberts Laimīte</p>
  </footer>
</template>

<script setup>
import upl from './components/upload.vue'
import ieraksts from './components/ieraksts.vue'
import ierakstuskaits from './components/ierakstuskaits.vue'
import search from './components/search.vue'
import { nextTick, ref } from 'vue'
import editcomp from './components/editcomp.vue'
</script>

<script>
const editpost = {
  idposts: 0
}
const editpostdata = ref(editpost)
const searchtext = ref('')
const showupload = ref(false)
export default {
  name: 'App',

  components: {
    upl,
    ieraksts,
    ierakstuskaits,
    editcomp
  },
  setup() {
    const ierskaits = ref(0)
    return {
      ierskaits,
      showupload
    }
  },
  mounted() {
    this.Updateview()
  },
  methods: {
    editfn(num) {
      editpostdata.value.idposts = num
      this.$refs.edit.show()
    },
    showuploadfn() {
      showupload.value = !showupload.value
    },
    async Updateview() {
      try {
        this.$refs.ierskaits.getpost()
      } catch (error) {
        console.log('ref get post err in appvue ')
      }
      await nextTick()
    },
    dosearch() {
      this.$refs.ier.dosearch()
    }
  }
}
</script>
