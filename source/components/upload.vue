<template>
  <div class="upload">
    <div>
      <form v-on:submit.prevent="uploadFile">
        <h1>Pievienot</h1>
        <p>Ieraksta virsraksts</p>
        <input
          type="text"
          v-model="uploadstate.title"
          placeholder="Ieraksta virsraksts"
          @change="uploadstate.title = $event.target.value"
        />
        <p>Ieraksta apraksts</p>
        <input
          type="text"
          v-model="uploadstate.pdesc"
          placeholder="Ieraksta apraksts"
          @change="uploadstate.pdesc = $event.target.value"
        />
        <div>
          <p>Ieraksta attels (var nepievienot)</p>
          <img id="preview" />
          <input id="fileupl" type="file" multiple @change="onMultipleFilesSelected" />
        </div>
        <div
          class="editgrid"
          v-if="uploadstate.preview.length > 0"
          v-for="item in uploadstate.preview"
        >
          <img id="preview" :src="item" />
          <button
            class="removebtn"
            type="button"
            @click="removefromupload(uploadstate.preview.indexOf(item))"
          >
            X
          </button>
        </div>

        <div>
          <button>Upload</button>
          <button @click="closebtn">Aizvert</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  name: 'upload',

  data() {
    const uploadstate = {
      title: '',
      pdesc: '',
      files: [],
      preview: []
    }
    return {
      uploadstate
    }
  },
  methods: {
    onMultipleFilesSelected(event) {
      let filearr = []
      for (let i = 0; i < event.target.files.length; i++) {
        filearr.push(event.target.files[i])
        this.renderpic(event.target.files[i])
      }
      this.uploadstate = { ...this.uploadstate, files: filearr, statenr: 1 }
    },

    renderpic(img) {
      let imgarr = this.uploadstate.preview
      const reader = new FileReader()
      reader.onload = (e) => {
        if (e.target) {
          imgarr.push(e.target.result) ?? ''
        }
      }
      reader.readAsDataURL(img)
      this.uploadstate = { ...this.uploadstate, preview: imgarr, statenr: 1 }
    },
    removefromupload(fileindex) {
      let pics = this.uploadstate.preview
      let filearr = this.uploadstate.files
      if (fileindex != -1) {
        pics.splice(fileindex, 1)
      }
      if (uploadstate.files != null) {
        if (fileindex != -1) {
          filearr.splice(fileindex, 1)
        }
      }
      this.uploadstate = { ...this.uploadstate, files: filearr, preview: pics, statenr: 1 }
    },

    uploadFile() {
      const formData = new FormData()
      formData.append('title', this.uploadstate.title)
      formData.append('pdesc', this.uploadstate.pdesc)
      if (this.uploadstate.files != null) {
        for (let i = 0; i < this.uploadstate.files.length; i++) {
          formData.append('file', this.uploadstate.files[i])
        }
      }
      fetch('http://localhost:3000/api/addpost', {
        method: 'post',
        body: formData
      }).catch(function (error) {
        console.log(error)
      })

      this.uploadstate = {
        ...this.uploadstate,
        title: '',
        pdesc: '',
        files: [],
        preview: [],
        statenr: 0
      }

      this.$emit('update')
    },
    closebtn() {
      this.$emit('closeupload')
    }
  }
}
</script>
