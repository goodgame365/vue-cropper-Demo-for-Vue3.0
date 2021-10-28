<template>
  <div class="home">

    <imgcropper :Cover="Cover" @uploadImgSuccess="getCover"></imgcropper>
  </div>
</template>

<script>
// @ is an alias to /src

import imgcropper from '../components/mecropper.vue'

export default {
  name: 'Home',
  data: function () {
    return {
      Cover: '/img/logo.82b9c7a5.png'
    }
  },
  components: {

    imgcropper
  },
  methods: {
    getCover (value) {
      this.Cover = value
    }
  }
}
</script>
