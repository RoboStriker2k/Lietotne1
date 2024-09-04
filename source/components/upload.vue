<template>
  <div class="upload">
    <div>
      <form v-on:submit.prevent="uploadFile">
        <h1>Pievienot</h1>
        <p>Ieraksta virsraksts</p>
        <input type="text" v-model="title" placeholder="Ieraksta virsraksts" />
        <p>Ieraksta apraksts</p>
        <input type="text" v-model="pdesc" placeholder="Ieraksta apraksts" />
        <div>
          <p>Ieraksta attels (var nepievienot)</p>
          <img id="preview" />
          <input id="fileupl" type="file" @change="onFileChange" />
        </div>
        <div>
          <button>Upload</button>
          <button @click="closebtn">Aizvert</button>
        </div>
      </form>
    </div>
  </div>
</template>
<script setup></script>

<script>
export default {
  name: 'upload',

  data() {
    return {
      title: '',
      pdesc: '',
      file: null
    }
  },
  methods: {
    onFileChange(event) {
      this.file = event.target.files[0]

      const reader = new FileReader()
      reader.onload = (e) => {
        let img = document.getElementById('preview')
        img.src = e.target.result
      }
      reader.readAsDataURL(this.file)
    },
    uploadFile() {
      const formData = new FormData()
      formData.append('title', this.title)
      formData.append('pdesc', this.pdesc)
      formData.append('file', this.file)
      fetch('http://localhost:3000/api/addpost', {
        method: 'post',
        body: formData
      }).catch(function (error) {
        console.log(error)
      })

      this.$emit('update')
    },
    closebtn() {
      this.$emit('closeupload')
    }
  }
}
</script>
