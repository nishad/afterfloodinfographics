<template>
  <transition name="fade">
    <b-row class="animated fadeIn">
      <b-col cols="12">
        <b-row class="justify-content-between">
          <b-col cols="12" md="6">
            <b-breadcrumb :items="breadcrumb_items"/>
          </b-col>
          <b-col cols="12" md="3">
            <input class="w-100 form-control search-input" v-model="search" placeholder="Search"/>
          </b-col>
        </b-row>
      </b-col>
      <b-col cols="12">
        <b-card-group columns>
          <nuxt-link :to="'/' + $route.params.language + '/' + doc['slug']" v-if="documents.length > 0" v-for="doc in documents" :key="doc.slug">
            <b-card :title="doc.title" v-if="doc.languages[$route.params.language].png && doc.languages[$route.params.language].png.length > 0" class="card-no-title">
              <div>
                <no-ssr>
                  <InfographicsCarousel :images="doc.languages[$route.params.language].png" />
                </no-ssr>
                <no-ssr>
                  <div class="card-footer" @click.native="preventDefault">
                    <div class="row">
                      <div class="col">
                        <p class="share-info">Share infographic</p>
                      </div>
                      <div class="col align-right">
                        <a class="sm-share-small" :href="'https://www.facebook.com/sharer.php?caption=After Flood Infographics on' + doc.title + '&description=After Flood Infographics on' + doc.title + '&u=https://infographics.afterflood.in/#/' + $route.params.language + '/' + doc['slug']" target="_blank">
                          <img src="/fb-icon-coloured.png">
                        </a>
                        <a class="sm-share-small" :href="'https://twitter.com/share?text=After Flood Infographics on' + doc.title + '&hashtags=AfterFlood, Infographics, KeralaFloods' + '&url=https://infographics.afterflood.in/#/' + $route.params.language + '/' + doc['slug']" target="_blank">
                          <img src="/twitter-icon-coloured.png">
                        </a>
                      </div>
                    </div>
                  </div>
                </no-ssr>
              </div>
            </b-card>
            <b-card :title="doc.title" v-else></b-card>
          </nuxt-link>
        </b-card-group>
      </b-col>
    </b-row>
  </transition>
</template>

<script>
import getGoogleImgUrl from '~/plugins/filters'
import getGoogleID from '~/plugins/filters'
import InfographicsCarousel from '~/components/InfographicsCarousel.vue'

export default {
  components: {
    InfographicsCarousel
  },
  data () {
    return {
      search: ""
    }
  },
  head () {
    let lang = this.$route.params.language
    lang = lang.replace(/\w\S*/g, (txt) => { return txt.charAt(0).toUpperCase() + txt.substr(1).toLowerCase();})
    return {
      'title': lang
    }
  },
  mounted () {
    if (this.$route.query.tag !== undefined && this.search === "") {
      this.search = this.$route.query.tag
    }
  },
  methods:{
    preventDefault: function(e){
      e.preventDefault();
    },
    getimgurl (drive_url) {
      try {
        return this.$options.filters.getGoogleImgUrl(this.$options.filters.getGoogleID(drive_url))
      } catch (err) {
        return  "/placeholder.png";
      }
    }
  },
  computed: {
    breadcrumb_items () {
      return [{
          text: 'Home',
          to: '/'
        }, {
          text: this.$route.params.language,
          active: true
        }]
    },
    documents () {
      let documents = []
      if (this.$store.state.index !== null) {
        documents = this.$store.state.index.filter((elem) => {
          let val = false
          if (elem.languages !== undefined && elem.languages !== null) {
            if (Object.keys(elem.languages).includes(this.$route.params.language)) {
              val = true
            }
          }
          return val
        })
      }
      if (this.search.length > 0) {
        let search_val = this.search.toLowerCase().replace(/\s+/g, '').trim()
        documents = documents.filter((elem) => {
          let search_slug = elem['slug'].replace(/\-/g, '')
          let slug_search = search_slug.includes(search_val)
          let tag_search = false
          let desc_search = false

          if (elem.tags != undefined) {
            tag_search = elem.tags.includes(this.search)
          }

          if (elem.description != undefined) {
            desc_search = elem.description.toLowerCase().includes(this.search)
          }
          return slug_search || tag_search || desc_search
        })
      }
      return documents
    }
  }
}
</script>

<style scoped lang="scss">
.card-no-title {
  position: relative;
  .card-title {
    display: none !important;
  }
  .card-body{
    padding: 0px;
  }
  .card-image-full {
    border-top: 0;
    filter: grayscale(100%);
    margin-bottom: 30px;
    transition: all 0.1s ease-in;
  }
  &:hover, &:focus, &:active {
    .card-image-full {
      filter: grayscale(0);
    }
  }
}
.card-footer{
  position: absolute;
  bottom: 0px;
  left: 0px;
  right: 0px;
  z-index: 999;
  background: #fff;
  border-top: 1px solid #ddd;
  padding: 15px 12px;
  &:last-child {
    border-radius: 0 !important;
  }
  .share-info{
    font-size: 0.9em;
    color: #999;
    margin: 0px;
  }

  .align-right{
    text-align: right;
  }
  .sm-share-small{
    display: inline-block;
    margin-right: 15px;

    img {
      max-width: 28px;
    }

    &:last-child{
      margin-right: 0px;
    }
  }
}
.search-input{
  border-radius: 4px;
}
.card-image-full {
  margin-bottom: 20px;
}
</style>
