<template>
  <div id="postcntbox">
    <p>Atrasto ierakstu skaits datubāze: {{ searchstates.postcount }}</p>
  </div>
  <Galleryview :gs="gs"></Galleryview>
  <div v-if="ieraksti.length == 0">Ierakstu nav</div>
  <div id="dzest">
    <button v-if="!searchstates.deletemarker & (ieraksti.length > 0)" v-on:click="deleteselector">
      Ierakstu atlase
    </button>
    <button v-if="searchstates.deletemarker" v-on:click="deletefunction">Dzest ierakstus</button>
    <button v-if="searchstates.deletemarker" v-on:click="deleteselector">Atcelt dzešanu</button>
  </div>
  <div class="iecontent">
    <div v-for="post in ieraksti">
      <div class="ieraksts" v-bind:id="post.idposts">
        <input
          v-if="searchstates.deletemarker"
          class="checkbox"
          type="CHECKBOX"
          v-bind:id="post.idposts"
        />
        <h1>{{ post.title }}</h1>
        <P>{{ post.pdesc }}</P>
        <img
          v-if="post.imgpath != null"
          :src="`http://localhost:3000/getfoto/${post.imgpath}`"
          alt="Image"
        />
        <Multiimgcomp v-if="post.imgarr != null" :imgarr="post.imgarr" />
        <div><button class="btn" v-on:click="editfn(post.idposts)">Labot</button></div>
        <div v-if="post.imgpath != null || post.imgarr != null">
          <button type="button" @click="setgallerystate({ postid: post.idposts, statenr: 1 })">
            Skatīt
          </button>
        </div>
      </div>
    </div>
  </div>
  <div id="pagenum">
    <button v-on:click="decpagebutton(), getposts(searchstates.postammount, searchstates.bookmark)">
      Back
    </button>
    <h1 id="pagecount">{{ searchstates.bookmark + 1 }}/{{ searchstates.pagecount + 1 }}</h1>
    <button v-on:click="incpagebutton(), getposts(searchstates.postammount, searchstates.bookmark)">
      Next
    </button>
  </div>
</template>

<script setup>
import Galleryview from './galleryview.vue'
import Multiimgcomp from './multiimgcomp.vue'
</script>
<script>
var deleteselection = []

export default {
  props: ['deleteselection'],
  components: { Multiimgcomp, Galleryview },
  expose: ['dosearch'],
  emits: ['update', 'editfn'],
  setup() {
    return {}
  },

  data() {
    const searchstates = {
      searchtext: '',
      postammount: 5,
      bookmark: 0,
      deletemarker: false,
      pagecount: 0,
      totalpostcount: 0,
      defaultsearchtext: '',
      defaultpostammount: 5,
      defaultbookmark: 0,
      postcount: 0
    }
    const gs = {
      idposts: 0,
      viewable: false
    }
    const ieraksti = []
    const imgsrc = ''

    return {
      ieraksti,
      imgsrc,
      searchstates,
      gs
    }
  },

  mounted() {
    this.update()

    this.getposts(this.searchstates.postammount, this.searchstates.bookmark)
  },

  methods: {
    setgallerystate(state) {
      this.gs = { ...this.gs, ...state }
    },
    editfn(id) {
      this.$emit('editfn', id)
    },
    async getposts(
      ammount = this.searchstates.defaultpostammount,
      offset = this.searchstates.defaultbookmark
    ) {
      let searchtextval = document.getElementById('searchbar').value

      const formData1 = new FormData()
      formData1.append('searchtext', searchtextval)
      formData1.append('ammount', ammount)
      formData1.append('offset', offset)
      fetch('http://localhost:3000/search', {
        method: 'post',
        body: formData1
      })
        .then((response) => response.json())
        .then((data) => {
          this.ieraksti = data.posts
          this.getpostcount(data.count)
        })
        .catch(function (error) {
          console.log(error)
        })

      this.$emit('update')
    },
    async getpostcount(count) {
      this.searchstates.postcount = count

      this.searchstates.pagecount =
        Math.ceil(this.searchstates.postcount / this.searchstates.postammount) - 1
    },

    incpagebutton() {
      if (this.searchstates.bookmark < this.searchstates.pagecount) {
        this.searchstates.bookmark++
      }
    },

    decpagebutton() {
      if (this.searchstates.bookmark > 0) {
        this.searchstates.bookmark--
      }
    },

    update() {
      this.getposts(this.searchstates.postammount, this.searchstates.bookmark)
      // this.$emit('update')
    },
    deleteselector() {
      this.searchstates.deletemarker = !this.searchstates.deletemarker
      // this.update()
    },

    deletefunction() {
      let selectledelate = document.querySelectorAll('input[type=checkbox]:checked')
      deleteselection = []
      for (var i = 0; i < selectledelate.length; i++) {
        document.getElementById(selectledelate[i].id).children[0].checked = false
        if (deleteselection.includes(selectledelate[i].id)) {
          continue
        } else {
          deleteselection.push(selectledelate[i].id)
        }
      }

      let deleteform = new FormData()
      if (deleteselection.length > 1) {
        for (var i = 0; i < deleteselection.length; i++) {
          deleteform.append('idlist', deleteselection[i])
          console.log('added' + deleteselection[i])
        }
        fetch('http://localhost:3000/api/deleteposts', {
          method: 'post',
          body: deleteform
        })
      } else if (deleteselection.length == 1) {
        deleteform.append('idlist', deleteselection[0])
        fetch('http://localhost:3000/api/deleteposts', {
          method: 'post',
          body: deleteform
        })
      }

      this.update()
      if (this.searchstates.bookmark > this.searchstates.pagecount) {
        this.searchstates.bookmark = this.searchstates.pagecount
      }
    },

    dosearch() {
      this.getposts()
    }
  }
}
</script>
