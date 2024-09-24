<template>
  <div v-if="PState.viewstatus" id="edit" class="editview">
    <div v-if="PState.ieraksti.length == 0">Ierakstu nav</div>
    <div v-if="PState.ieraksti.length != 0" v-for="item in PState.ieraksti">
      <div>
        <h1>Pievienot</h1>
        <div>
          <label for="edittitle">Ieraksta virsraksts</label>
          <input id="edittitle" type="text" :placeholder="[[item.title]]" @change="updatetitle" />
        </div>
        <div>
          <label for="editdesc">Ieraksta apraksts</label>
          <input id="editdesc" type="text" :placeholder="[[item.pdesc]]" @change="updatedesc" />
        </div>
        <div>
          <p>Pievienot attēlus</p>
          <img id="editpreview" />
          <input id="editupl" type="file" multiple @change="onMultipleFilesSelected" />
        </div>
      </div>
      <div class="editier">
        <h1>Priekšskats</h1>
        <div class="editieraksts" id="{{ item.idposts }}">
          <h1 id="previewtitle">{{ PState.newtitle }}</h1>
          <p id="previewdesc">{{ PState.newpdesc }}</p>
          <img
            v-if="item.imgpath != null"
            id="previewimg"
            :src="`http://localhost:3000/getfoto/?file=${item.imgpath}`"
          />
          <Multiimgcomp v-if="item.imgarr != null" :imgarr="item.imgarr" :editstatus="true" />
          <div class="editgrid" v-if="PState.preview.length > 0" v-for="item in PState.preview">
            <img id="preview" :src="item" />
            <button
              class="removebtn"
              type="button"
              @click="removefromupload(PState.preview.indexOf(item))"
            >
              X
            </button>
          </div>
        </div>
      </div>

      <div>
        <button type="button" @click="removechecked()">
          Dzēst atzīmētos attēlus neatgriezeniski
        </button>
        <button type="button" @click="editfn()">Labot ierakstu</button>
        <button type="button" @click="hide()">Atcelt</button>
        <button type="button" @click="hide()">Aizvert</button>
      </div>
    </div>
  </div>
</template>
<script>
import Multiimgcomp from './multiimgcomp.vue'

export default {
  name: 'editcomp',
  props: ['editid'],
  components: { Multiimgcomp },
  emits: ['updateview'],

  setup() {
    return {}
  },
  data() {
    const PState = {
      ieraksti: [],
      newtitle: '',
      newpdesc: '',
      files: [],
      preview: [],
      viewstatus: false
    }
    return {
      PState
    }
  },

  mounted() {
    this.getpost()
  },
  methods: {
    show() {
      this.PState.viewstatus = true
      this.getpost()
    },
    hide() {
      this.PState.viewstatus = false
      this.PState = {
        ieraksti: [],
        newtitle: '',
        newpdesc: '',
        files: [],
        preview: [],
        viewstatus: false
      }
      this.editid.idposts = null
    },
    updatetitle(event) {
      this.PState.newtitle = event.target.value
    },
    updatedesc(event) {
      this.PState.newpdesc = event.target.value
    },

    editfn() {
      console.log(
        this.PState.ieraksti.title,
        this.PState.ieraksti.pdesc,
        this.PState.newtitle,
        this.PState.newpdesc
      )
      let formdata = new FormData()
      if (this.PState.ieraksti.title != this.PState.newtitle) {
        formdata.append('title', this.PState.newtitle)
      }
      if (this.PState.ieraksti.pdesc != this.PState.newpdesc) {
        formdata.append('pdesc', this.PState.newpdesc)
      }
      if (this.PState.files != null) {
        for (let i = 0; i < this.PState.files.length; i++) {
          formdata.append('file', this.PState.files[i])
        }
      }
      formdata.append('idpost', this.editid.idposts)
      formdata.append('replaceflag', 'false')
      formdata.append('removeflag', 'false')

      fetch('http://' + window.location.hostname + '3000/api/editpost/', {
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
      let id = this.editid.idposts
      fetch('http://' + window.location.hostname + ':3000/api/getpost/?postiid=${id}', {
        method: 'GET'
      })
        .then((response) => response.json())
        .then((data) => {
          this.PState.ieraksti = data.posts
          // noņem references no iegūtā datu objekta
          this.PState.newtitle = toString(this.PState.ieraksti.title)
          this.PState.newpdesc = toString(this.PState.ieraksti.pdesc)
        })
        .catch((error) => {
          console.error('Error:', error)
        })
      if (this.PState.ieraksti.length > 0) {
        this.PState.viewstatus = true
      }
    },

    onMultipleFilesSelected(event) {
      let filearr = []
      for (let i = 0; i < event.target.files.length; i++) {
        filearr.push(event.target.files[i])
        this.renderpic(event.target.files[i])
      }
      this.PState.files = filearr
    },

    removefromupload(fileindex) {
      let pics = this.PState.preview
      let filearr = this.PState.files
      if (fileindex != -1) {
        pics.splice(fileindex, 1)
        this.PState.preview = pics
      }

      if (this.PState.files != null) {
        if (fileindex != -1) {
          filearr.splice(fileindex, 1)
          this.PState.files = filearr
        }
      }
    },
    renderpic(img) {
      let imgarr = this.PState.preview
      const reader = new FileReader()
      reader.onload = (e) => {
        if (e.target) {
          imgarr.push(e.target.result) ?? ''
        }
      }
      reader.readAsDataURL(img)
      this.PState.preview = imgarr
    },

    removechecked() {
      console.log('removing checked')
      let selectledelate = document.querySelectorAll('input[type=checkbox]:checked')
      let deleteform = new FormData()
      console.log('ran', selectledelate)
      let removeflag = true
      let replaceflag = false
      let imgarr = []
      if (selectledelate.length === 0) {
        return
      } else if (selectledelate.length > 1) {
        for (let i = 0; i < selectledelate.length; i++) {
          const elem = document.getElementById(selectledelate[i].id)
          if (elem) {
            removeflag = true
            if (imgarr.includes(selectledelate[i].id)) {
              continue
            } else if (elem.classList.contains('multiimgcheck')) {
              imgarr.push(selectledelate[i].id)
            } else {
              imgarr.push(selectledelate[i].id)
              deleteform.append('imgpath', selectledelate[i].id)
            }
          }
        }
      } else if (selectledelate.length == 1) {
        imgarr.push(selectledelate[0].id)
      }

      if (imgarr != null) {
        console.log(JSON.stringify(imgarr))
        deleteform.append('imgarr', JSON.stringify(imgarr))
      }
      deleteform.append('replaceflag', replaceflag.toString())
      deleteform.append('removeflag', removeflag.toString())
      deleteform.append('idpost', this.editid.idposts)
      fetch('http://' + window.location.hostname + ':3000/api/editpost/', {
        method: 'POST',
        body: deleteform
      })
        .then((response) => response.json())
        .then((data) => {
          console.log(data)
        })
        .catch((error) => {
          console.error('Error:', error)
        })
    }
  }
}
</script>
