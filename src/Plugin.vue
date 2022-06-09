<template>
  <div>
    <div class="uk-form-row">
      <input type="text" placeholder="https://www.youtube.com/watch?v=4l" v-model="model.url" class="uk-width-1-1">
      <div v-if="fetchingEmbed" class="uk-text-small uk-text-muted uk-margin-small-top">
        <i class="uk-icon-spinner uk-icon-spin"></i>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  mixins: [window.Storyblok.plugin],
  data() {
    return {
      fetchingEmbed: false,
    }
  },
  methods: {
    initWith() {
      return {
        // needs to be equal to your storyblok plugin name
        plugin: 'embed',
        url: '',
        oembed: null,
        error: null,
        description: ''
      }
    },
  },
  watch: {
    'model.url': {
      handler: function (url) {
        const that = this
        if (this.fetchingEmbedTimer) {
          window.clearTimeout(this.fetchingEmbedTimer)
        }
        if (url != null && url.length > 0) {
          this.fetchingEmbedTimer = window.setTimeout(() => {
            this.fetchingEmbed = true
            fetch(`https://oembed-service.vercel.app/api/resolve?url=${url}&secret=${that.token}`)
              .then(response => response.json())
              .then(data => {
                this.fetchingEmbed = false
                that.model.oembed = data
              })
              .catch(error => {
                this.fetchingEmbed = false
                that.model.oembed = null
                console.error("Error fetching embed", error)
              })
          }, 500)
        } else {
          that.model.oembed = null
        }
      }
    },
    'model': {
      handler: function (value) {
        this.$emit('changed-model', value);
      },
      deep: true
    }
  }
}
</script>
