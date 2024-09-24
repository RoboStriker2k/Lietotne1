<template>
  <div id="gallerymarker" v-if="gstate.view">
    <div class="gallery">
      <div id="galleryimg">
        <img :src="`${imgurl + gstate.currimg}`" />
      </div>
      <div class="gallerynav">
        <button id="prev" @click="previmg()">{{ '<' }}</button>
        <div id="currimg"><img id="currimg" :src="`${imgurl + gstate.currimg}`" /></div>
        <button id="next" @click="nextimg">{{ '>' }}</button>
        <button style="float: 'right'" @click="close">iziet</button>
      </div>
      <div v-if="gstate.IER.imgarr != null">
        <div class="album">
          <div v-for="img in gstate.IER.imgarr.images">
            <img :src="`${imgurl + img}`" v-on:click="setgstate({ statenr: 1, currimg: img })" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script setup></script>

<script>
export default {
  name: 'galleryview',
  props: ['gs'],
  emits: ['resetgallery'],
  data() {
    const imgurl = 'http://' + window.location.hostname + ':3000/getfoto/?file='
    const gstate = {
      IER: {
        imgarr: {
          images: []
        }
      },
      pw: {
        images: ['']
      },
      statenr: -1,
      currimg: '',
      imageindex: 0,
      view: false
    }

    return {
      imgurl,
      gstate
    }
  },
  mounted() {
    this.getpost()
  },
  methods: {
    show() {
      this.getpost()
      this.gstate.view = true
    },
    setgstate(state) {
      this.gstate = { ...this.gstate, ...state }
    },
    nextimg() {
      let index = this.gstate.imageindex
      if (index < this.gstate.pw.images.length - 1) {
        this.setgstate({
          ...this.gstate,
          imageindex: index + 1,
          currimg: this.gstate.pw.images[index + 1]
        })
      }
    },
    previmg() {
      let index = this.gstate.imageindex
      if (index > 0) {
        this.setgstate({
          ...this.gstate,
          imageindex: index - 1,
          currimg: this.gstate.pw.images[index - 1]
        })
      }
    },
    getpost() {
      if (this.gs.idposts != 0)
        fetch(`http://${window.location.hostname}:3000/api/getpost/?postiid=${this.gs.idposts}`, {
          method: 'GET'
        })
          .then((response) => response.json())
          .then((data) => {
            this.gstate = {
              ...this.gstate,
              IER: data.posts[0],
              pw: data.posts[0].imgarr,
              currimg: data.posts[0].imgarr.images[0]
            }
            if (this.gstate.IER.length > 0) {
              this.gstate.statenr = 1
            }
          })
          .catch((error) => {
            console.log(error)
          })
    },
    close() {
      this.gstate = {
        ...this.gstate,
        statenr: -1,
        imageindex: 0,
        currimg: this.gstate.pw.images[0],
        view: false
      }
      this.resetgallery.emit()
    }
  }
}
</script>
