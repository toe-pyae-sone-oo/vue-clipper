<template>
  <div id="app">
    <clipper
      :width="image.width"
      :height="image.height"
      @image-cropped="showImage"
    >
      <img 
        alt="Wallpaper" 
        src="./assets/wallpaper.jpg" 
        :width="image.width" 
        :height="image.height"
      />
    </clipper>
    <div v-if="croppedImage">
      <p>Result:</p>
      <img :src="croppedImage" />
    </div>
  </div>
</template>

<script>
import clipper from './lib/clipper';

export default {
  name: 'App',

  components: {
    clipper,
  },

  data() {
    return {
      image: {
        width: 1424,
        height: 945,
      },

      croppedImage: undefined,
    };
  },

  methods: {
    showImage(img) {
      this.croppedImage = img.url;
    }
  },
}
</script>

<style>
</style>
