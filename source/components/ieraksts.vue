<template>
  <div id="postcntbox">
    <p>Atrasto ierakstu skaits datubāze: {{ atrastoierakstuskaits }}</p>
  </div>
  <div v-if="ieraksti.length == 0">Ierakstu nav</div>
  <div id="dzest">
    <button v-if="!deletemarker & (ieraksti.length > 0)" v-on:click="deleteselector">
      Ierakstu atlase
    </button>
    <button v-if="deletemarker" v-on:click="deletefunction">Dzest ierakstus</button>
    <button v-if="deletemarker" v-on:click="deleteselector">Atcelt dzešanu</button>
  </div>
  <div class="iecontent">
    <div v-for="post in ieraksti">
      <div class="ieraksts" v-bind:id="post.idposts">
        <input v-if="deletemarker" class="checkbox" type="CHECKBOX" v-bind:id="post.idposts" />
        <h1>{{ post.title }}</h1>
        <P>{{ post.pdesc }}</P>
        <P v-if="DEBUGVAL">{{ post.idposts }}</P>

        <img
          v-if="post.imgpath != null"
          :src="`http://localhost:3000/getfoto/${post.imgpath}`"
          alt="Image"
        />
        <div><button class="btn" v-on:click="editfn(post.idposts)">Labot</button></div>
      </div>
    </div>
  </div>
  <div id="pagenum">
    <button v-on:click="decpagebutton(), getposts(getpostammount, postbookmark)">Back</button>
    <h1 id="pagecount">{{ postbookmark + 1 }}/{{ pagecount + 1 }}</h1>
    <button v-on:click="incpagebutton(), getposts(getpostammount, postbookmark)">Next</button>
  </div>
</template>

<script setup>
import axios from 'axios'
</script>
<script>
let getpostammount = 5
var postbookmark = 0
var postcount = 0
var pagecount = 0
var deletemarker = false
var deleteselection = []
var DEBUGVAL = false
var defaultsearchtext = ''

export default {
  props: ['deletemarker', 'deleteselection'],
  expose: ['dosearch'],
  emits: ['update', 'editfn'],
  setup() {
    return {}
  },

  data() {
    const ieraksti = []
    const imgsrc = ''
    let atrastoierakstuskaits = 0
    return {
      ieraksti,
      imgsrc,
      atrastoierakstuskaits
    }
  },

  mounted() {
    this.getpostcount()
    this.update()

    this.getposts(getpostammount, postbookmark)
  },

  methods: {
    editfn(id) {
      this.$emit('editfn', id)
    },
    async getposts(ammount, offset) {
      if (!ammount) {
        ammount = getpostammount
      }
      if (!offset) {
        offset = 0
      }
      let searchtextval = document.getElementById('searchbar').value

      if (searchtextval == '') {
        const response = await axios.get(
          `http://localhost:3000/api/getposts?ammount=${ammount}&offset=${offset}`
        )
        this.ieraksti = response.data.posts
      } else {
        const formData1 = new FormData()
        formData1.append('searchtext', searchtextval)
        formData1.append('ammount', ammount)
        formData1.append('offset', offset)
        console.log(searchtextval)
        const response2 = axios
          .post('http://localhost:3000/search', formData1)

          .catch(function (error) {
            console.log(error)
          })
          .then(function (response) {
            console.log(response)
            return response
          })
          .then((response) => {
            this.ieraksti = response.data.posts
            this.atrastoierakstuskaits = response.data.count
            this.getpostcount(response.data.count)
          })
        this.$emit('update')
      }
    },
    async getpostcount(count) {
      if (!count) {
        const response = await axios.get(`http://localhost:3000/api/postscount/`)
        postcount = response.data.posts[0].postscount
      }

      if (count) {
        postcount = count
      }
      pagecount = Math.ceil(postcount / getpostammount) - 1
    },

    incpagebutton() {
      if (postbookmark < pagecount) {
        postbookmark++
      }
    },

    decpagebutton() {
      if (postbookmark > 0) {
        postbookmark--
      }
    },

    update() {
      this.getpostcount()
      this.getposts(getpostammount, postbookmark)

      this.$emit('update')
    },
    deleteselector() {
      deletemarker = !deletemarker
      this.update()
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

        axios.post(`http://localhost:3000/api/deleteposts/`, deleteform)
      } else if (deleteselection.length == 1) {
        deleteform.append('idlist', deleteselection[0])
        axios.post(`http://localhost:3000/api/deleteposts/`, deleteform)
      }

      this.update()
      if (postbookmark > pagecount) {
        this.postbookmark = pagecount
      }
    },

    dosearch() {
      this.getposts()
    }
  }
}
</script>
