<template>
  <div>
    <section class="mb-5 grey">
      <v-img
        :src="image"
        height="300px"
        contain
        gradient="to top, rgba(0, 0, 0,.33), rgba(0, 0, 0,.33)"
      >
        <v-row align="center" class="lightbox white--text pa-2 fill-height">
          <v-col>
            <h3 v-if="attribution" class="text-center mb-5">
              {{ attribution }}
            </h3>
            <h1 class="display-1 text-center">
              <b>{{ title }}</b>
            </h1>
          </v-col>
        </v-row>
      </v-img>
    </section>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'

@Component
export default class Page extends Vue {
  head() {
    return {
      titleTemplate: null,
    }
  }

  image: string = this.$store.state.thumbnail ? this.$store.state.thumbnail : ''

  get title(): string {
    return 'ジャパンサーチ利活用スキーマを用いた機械分類サンプル'
  }

  get thumbnail(): string {
    return this.$store.state.thumbnail
  }

  get attribution(): string {
    return this.$store.state.attribution
  }

  get description(): string {
    return this.$store.state.description
  }
}
</script>
