<template>
  <div v-if="viewstatus" id="edit" class="editview">
    <div v-if="CIER.length == 0">Ierakstu nav</div>
    <div v-if="CIER.length != 0" v-for="item in CIER">
      <div>
        <h1>Pievienot</h1>
        <p>Ieraksta virsraksts</p>
        <input id="edittitle" type="text" :placeholder="[[item.title]]" @change="updatetitle" />
        <p>Ieraksta apraksts</p>
        <input id="editdesc" type="text" :placeholder="[[item.pdesc]]" @change="updatedesc" />
        <div>
          <p>Ieraksta attels (var nepievienot)</p>
          <img id="editpreview" />
          <input id="editupl" type="file" @change="onFileSelected" />
        </div>
      </div>

      <div class="editier">
        <h1>Priekšskats</h1>
        <div class="ieraksts" id="{{ item.idposts }}">
          <h1 id="previewtitle">{{ item.title }}</h1>
          <p id="previewdesc">{{ item.pdesc }}</p>
          <img
            v-if="item.imgpath != null"
            id="previewimg"
            :src="`http://localhost:3000/getfoto/${item.imgpath}`"
          />
        </div>
      </div>

      <div>
        <button type="button" @click="editfn()">Labot ierakstu</button>
        <button type="button" @click="hide()">Atcelt</button>
        <button type="button" @click="hide()">Aizvert</button>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  name: 'editcomp',
  props: ['Posttatus'],
  components: {},
  emits: ['updateview'],

  setup() {
    return {}
  },
  data() {
    const IER = []
    const CIER = []
    const file = null
    const viewstatus = false
    return {
      IER,
      CIER,
      file,
      viewstatus
    }
  },

  mounted() {
    this.getpost()
  },
  methods: {
    show() {
      this.Posttatus.status = true
      this.viewstatus = true
      this.getpost()
    },
    hide() {
      this.viewstatus = false
    },
    exit() {
      this.viewstatus = false
      this.$emit('updateview')
    },
    updatetitle(event) {
      this.CIER[0].title = event.target.value
    },
    updatedesc(event) {
      this.CIER[0].pdesc = event.target.value
    },

    editfn() {
      let oldtitle = this.IER[0].title
      let oldpdesc = this.IER[0].pdesc
      let oldimgpath = this.IER[0].imgpath
      let newtitle = this.CIER[0].title
      let newpdesc = this.CIER[0].pdesc
      console.log(oldtitle, oldpdesc, oldimgpath, newtitle, newpdesc)
      let formdata = new FormData()
      if (oldtitle != newtitle) {
        formdata.append('title', newtitle)
      }
      if (oldpdesc != newpdesc) {
        formdata.append('pdesc', newpdesc)
      }
      if (this.file != null) {
        formdata.append('file', this.file)
      }
      formdata.append('idpost', this.Posttatus.idposts.toString())

      fetch(`http://localhost:3000/api/editpost/`, {
        method: 'POST',
        body: formdata
      })
        .then((response) => response.json())
        .then((data) => {
          console.log(data)
        })
        .catch((error) => {
          console.error('Error:', error)
        })
    },

    getpost() {
      let id = this.Posttatus.idposts
      fetch(`http://localhost:3000/api/getpost/?postiid=${id}`, {
        method: 'GET'
      })
        .then((response) => response.json())
        .then((data) => {
          this.IER = data.posts
          /// nevareja vienkarši piešķirt data.posts jo  tas pieķšira refenci uz data.posts nevis kopeja datus
          this.CIER = JSON.parse(JSON.stringify(this.IER))
        })
        .catch((error) => {
          console.error('Error:', error)
        })
      this.Posttatus.status = true
    },

    onFileSelected(event) {
      this.file = event.target.files[0]
      if (this.file) {
        let img = document.getElementById('previewimg')
        const reader = new FileReader()
        reader.onload = (e) => {
          if (e.target) {
            img.src = e.target.result ?? ''
          }
        }
        reader.readAsDataURL(this.file)
      }
    }
  }
}
</script>
