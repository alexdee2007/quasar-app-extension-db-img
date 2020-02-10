<template>
  <div :style="{cursor: loaded ? 'zoom-in' : 'default', height, width}" :class="{'q-bordered rounded-borders': bordered}">
    <q-img
      v-bind="$props"
      :src="imgSrc"
      ref="img"
      v-viewer="viewerOptions"
      @load="loaded=true"
      @click.stop.prevent
      class="fit"
      >
      <template>
        <slot></slot>
      </template>
      <img :src="imgSrc" class="fit" style="opacity: 0; object-fit: contain;">
    </q-img>
  </div>
</template>

<script>

  import minioApi from 'src/api/minio';

  export default {
    name: 'DbImg',
    props: {
      height: {
        type: String,
        default: '100%'
      },
      width: {
        type: String,
        default: '100%'
      },
      bordered: Boolean,
      url: String,
      fileName: String,
      srcset: String,
      sizes: String,
      alt: String,
      placeholderSrc: String,
      basic: Boolean,
      contain: Boolean,
      position: String,
      ratio: [String, Number],
      transition: String,
      noDefaultSpinner: Boolean,
      spinnerColor: String,
      spinnerSize: String,
      imgClass: [Array, String, Object]
    },
    data() {
      return {
        imgSrc: '',
        loaded: false,
        viewerOptions: {
          navbar: false,
          zIndex: 99999,
          url: () => this.imgSrc,
          title: (image, imageData) => `${image.alt} (${imageData.naturalWidth} × ${imageData.naturalHeight})`,
          toolbar: {
            zoomIn: {size: 'large'},
            zoomOut: {size: 'large'},
            oneToOne: {size: 'large'},
            reset: {size: 'large'},
            play: {size: 'large'},
            rotateLeft: {size: 'large'},
            rotateRight: {size: 'large'},
            flipHorizontal: {size: 'large'},
            flipVertical: {size: 'large'}
          }
        }
      }
    },
    methods: {
      async getImgSrc() {
        try {
          this.$refs.img.isLoading = true;
          this.imgSrc = await minioApi.getPresignedUrlForGet(this.fileName);
        } catch (err) {
          console.error(err);
        } finally {
          this.$refs.img && (this.$refs.img.isLoading = false);
        }
      }
    },
    mounted() {
      if (this.url) {
        this.imgSrc = this.url;
      } else {
        this.getImgSrc();
      }

    }
  }
</script>

<style>
  .viewer-backdrop {
    background-color:rgba(0, 0, 0, 0.7) !important;
  }
  .viewer-button {
    background-color: rgba(0, 0, 0, 0.9) !important;
    height: 90px !important;
    width: 90px !important;
  }
  .viewer-button::before {
    bottom: 20px !important;
    left: 25px !important;
  }
</style>
