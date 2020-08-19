<template>
  <div>
    <v-app-bar id="head" flat>
      <v-toolbar-title>
        <nuxt-link
          :to="localePath({ name: 'index', query: { u: $route.query.u } })"
          style="color: inherit; text-decoration: inherit;"
        >
          {{ title }}
        </nuxt-link>
      </v-toolbar-title>

      <v-spacer></v-spacer>

      <v-menu offset-y>
        <template v-slot:activator="{ on }">
          <v-btn depressed btn v-on="on">
            <v-icon class="mr-2">mdi-translate</v-icon>
            <template v-if="!isMobile()">
              {{ $i18n.locale == 'ja' ? '日本語' : 'English' }}
              <v-icon class="ml-2">mdi-menu-down</v-icon>
            </template>
          </v-btn>
        </template>

        <v-list>
          <v-list-item :to="switchLocalePath('ja')">
            <v-list-item-title>日本語</v-list-item-title>
          </v-list-item>
          <v-list-item :to="switchLocalePath('en')">
            <v-list-item-title>English</v-list-item-title>
          </v-list-item>
        </v-list>
      </v-menu>
    </v-app-bar>
  </div>
</template>

<script lang="ts">
import { Vue, Component, Watch } from 'nuxt-property-decorator'

@Component
export default class Header extends Vue {
  trigger(event: any) {
    // 日本語入力中のEnterキー操作は無効にする
    if (event.keyCode !== 13) return

    this.search()
  }

  width: number = window.innerWidth
  height: number = window.innerHeight

  get title(): string {
    return 'ジャパンサーチ利活用スキーマを用いた機械分類サンプル'
  }

  handleResize() {
    // resizeのたびにこいつが発火するので、ここでやりたいことをやる
    this.width = window.innerWidth
    this.height = window.innerHeight
  }

  mounted() {
    window.addEventListener('resize', this.handleResize)
  }

  beforeDestroy() {
    window.removeEventListener('resize', this.handleResize)
  }

  drawer: boolean = false
  fixed: boolean = false

  keywordStr: string = ''
  keywords: string[] = []

  dialog: boolean = false

  advanced: any = {}

  get dataType(): string {
    if (this.$store.state.json) {
      return this.$store.state.json['@type']
    } else {
      return ''
    }
  }

  creators: any = [
    {
      value: '',
      text: 'All',
    },
    {
      value: 'Giovanni Battista Piranesi',
      text: 'Giovanni Battista Piranesi',
    },
    {
      value: 'Francesco Piranesi',
      text: 'Francesco Piranesi',
    },
  ] // 'All', 'Giovanni Battista Piranesi', 'Francesco Piranesi'

  // 保留。queryStoreを使いたい。
  @Watch('$route', { deep: true, immediate: true })
  watchRoute(val: any) {
    let keywords: any = val.query.keyword
    if (keywords) {
      keywords = this.$utils.convert2arr(keywords)
      this.keywordStr = this.$utils.formatArrayValue(keywords, ' ')
    } else {
      this.keywordStr = ''
    }
    /*
    const keywords: any = val.query.keyword
    if (keywords) {
      this.keywords = this.$utils.convert2arr(keywords)
      this.keywordStr = this.$utils.formatArrayValue(this.keywords, ' ')
    } else {
      this.keywordStr = ''
    }
    */
  }

  search() {
    let keywordStr = this.keywordStr

    if (!keywordStr) {
      keywordStr = ''
    }

    let keywords
    if (keywordStr.startsWith('"') && keywordStr.endsWith('"')) {
      keywords = [
        {
          label: 'keyword',
          value: keywordStr,
        },
      ]
    } else {
      keywords = this.$utils.splitKeyword(keywordStr)
    }

    // push 処理
    const query: any = JSON.parse(JSON.stringify(this.$route.query))

    // キーワードは初期化してみる
    delete query.keyword

    for (let i = 0; i < keywords.length; i++) {
      const obj: any = keywords[i]
      const label = obj.label
      const value = obj.value

      if (!query[label]) {
        query[label] = []
      } else if (!Array.isArray(query[label])) {
        query[label] = [query[label]]
      }

      if (!query[label].includes(value)) {
        query[label].push(value)
      }
    }
    // query.keyword = keywords
    query.from = 0

    this.$router.push(
      this.localePath({
        name: 'search',
        query,
      }),
      () => {},
      () => {}
    )
  }

  /*
  advancedSearch() {
    const advanced = this.advanced
    const query: any = Object.assign({}, this.$route.query)
    for (const term in advanced) {
      const value: string = advanced[term].trim()
      if (value === '') {
        if (query[term]) {
          delete query[term]
        }
      } else {
        query[term] = value
      }
    }

    const keywordStr = this.keywordStr

    if (keywordStr) {
      const keywords = this.$utils.splitKeyword(keywordStr)
      query.keyword = keywords
    }

    query.from = 0

    this.$router.push(
      this.localePath({
        name: 'search',
        query,
      }),
      () => {},
      () => {}
    )

    this.dialog = false
  }
  */

  isMobile() {
    if (
      /Android|webOS|iPhone|iPad|iPod|BlackBerry|IEMobile|Opera Mini/i.test(
        navigator.userAgent
      )
    ) {
      return true
    } else {
      return false
    }
  }
}
</script>
