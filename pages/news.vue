<template>
  <div class="page-container mb-3">
    <br /><img class="page-header" src="~/assets/image/page/banners/news.png" />
    <div
      class="row mt-2 page-container"
      v-for="news in this.$store.state.news"
      :key="news.id"
    >
      <p class="subheading freedom center space up margin">{{ news.title }}</p>
      <br />
      <img class="news" :src="newsImage(news)" v-if="news.img" />
      <br />
      <span class="base" v-html="news.content"></span>
    </div>
  </div>
</template>

<script>
export default {
  head() {
    return {
      title: 'Cooperacy - News'
    }
  },
  async fetch({ store, params }) {
    await store.dispatch('getNewsAction', '')
  },
  methods: {
    newsImage(img) {
      if (img.img) {
        return require('~/assets/image/news/' + img.img)
      }
    }
  }
}
</script>
