<template>
  <div id="multiimgdisplay">
    <div id="multiimg">
      <div v-for="img of imgarr.images">
        <img :src="`${imgurl + img}`" />

        <input class="multiimgcheck" v-if="editstatus" type="checkbox" :id="`${img}`" />
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: 'Multiimgcomp',
  props: {
    imgarr: Object,
    editstatus: Boolean
  },
  data() {
    const imgurl = 'http://localhost:3000/getfoto/?file='
    return { imgurl }
  }
}
</script>
