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
      <galleryview :gs="showgallery" ref="gview" @resetgallery="resetgs"></galleryview>
      <div class="content-header">
        <ierakstuskaits ref="ierskaits" />
        <h1>Lietotnes ieraksti</h1>
      </div>
      <div id="dyncontent">
        <ieraksts @update="Updateview" @editfn="editfn" ref="ier" @gs="gs" />
        <editcomp :editid="editpostdata" ref="edit" @update="Updateview" />
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
import galleryview from './components/galleryview.vue'
</script>

<script>
const editpost = {
  idposts: 0
}
const gstate = {
  idposts: 0
}
const editpostdata = ref(editpost)
const searchtext = ref('')
const showupload = ref(false)
const ierskaits = ref(0)
const showgallery = ref(gstate)
export default {
  name: 'App',

  components: {
    upl,
    ieraksts,
    ierakstuskaits,
    editcomp,
    galleryview
  },
  setup() {
    return {
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
      this.$refs.ierskaits.getpost()
      await nextTick()
    },
    dosearch() {
      this.$refs.ier.dosearch()
    },
    gs(num) {
      showgallery.value.idposts = num
      this.$refs.gview.show()
    },
    resetgs() {
      showgallery.value.idposts = 0
    }
  }
}
</script>
