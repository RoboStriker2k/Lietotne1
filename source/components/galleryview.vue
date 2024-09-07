<template>
  <div id="gallerymarker">
    <div class="gallery" v-if="gstate.statenr != -1">
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
        <div class="album" v-for="img in gstate.IER.imgarr.images">
          <div>
            <img :src="`${imgurl + img}`" v-on:click="setgstate({ statenr: 1, currimg: img })" />
          </div>
        </div>
      </div>
      >
    </div>
  </div>
</template>
<script setup></script>

<script>
export default {
  name: 'galleryview',
  props: {
    gs: {
      idposts: 390,
      viewable: false
    }
  },

  setup() {
    return {}
  },

  data() {
    const imgurl = 'http://localhost:3000/getfoto/?file='
    const gstate = {
      IER: {
        imgarr: {
          images: []
        }
      },
      pw: {
        images: ['']
      },
      statenr: 0,
      currimg: '',
      imageindex: 0
    }
    const test = {
      lol: 0
    }
    return {
      imgurl,
      gstate,
      lol
    }
  },
  methods: {
    setgstate(state) {
      gstate = { ...this.gstate, ...state }
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
      fetch(`http://localhost:3000/api/getpost/?postiid=${this.gs.idposts}`, {
        method: 'GET'
      })
        .then((response) => response.json())
        .then((data) => {
          this.gstate = {
            ...this.gstate,
            IER: data.posts[0],
            statenr: 1,
            pw: data.posts[0].imgarr,
            currimg: data.posts[0].imgarr.images[0]
          }
          this.gs.viewable = true
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
        currimg: this.gstate.pw.images[0]
      }
      this.resetgalery.emit()
    }
  }
}
</script>
