<template>
  <div>
    <v-container>
      <v-expansion-panels class="my-10">
        <v-expansion-panel>
          <v-expansion-panel-header class="primary white--text">
            {{ $t('example') }}
          </v-expansion-panel-header>
          <v-expansion-panel-content>
            <v-row>
              <v-col
                v-for="(obj, index) in samples"
                :key="index"
                cols="3"
                sm="2"
              >
                <v-img
                  contain
                  max-height="200px"
                  :src="obj.url"
                  @click="
                    uploadImageUrl = obj.url
                    text = obj.text
                    title = obj.title
                  "
                />
              </v-col>
            </v-row>
          </v-expansion-panel-content>
        </v-expansion-panel>
      </v-expansion-panels>

      <v-card class="my-10">
        <v-card-title class="primary white--text">
          {{ $t('検索条件') }}
        </v-card-title>
        <v-container>
          <v-row>
            <v-col cols="12" sm="4">
              <v-card outlined class="mb-5">
                <v-card-title class="grey lighten-2">
                  {{ $t('title') }}
                </v-card-title>

                <v-container>
                  <v-text-field
                    v-model="title"
                    placeholder="桐壺"
                    outlined
                  ></v-text-field>
                </v-container>
              </v-card>
            </v-col>

            <v-col cols="12" sm="4">
              <v-card outlined class="mb-5">
                <v-card-title class="headline grey lighten-2">
                  {{ $t('text') }}
                </v-card-title>

                <v-container>
                  <v-textarea
                    v-model="text"
                    placeholder="書名：源氏物語
書名ヨミ：ゲンジ モノガタリ
請求記号：A00:6587
一般注記：青洲文庫書名は帙題簽による
巻次は題簽による
責任表示は『日本古典籍総合目録データベース』による
朱筆書き入れあり
虫損あり
旧請求記号 E23:48"
                    outlined
                    auto-grow
                  ></v-textarea>
                </v-container>
              </v-card>
            </v-col>

            <v-col cols="12" sm="4">
              <v-card outlined class="mb-5">
                <v-card-title class="headline grey lighten-2">
                  {{ $t('image') }}
                </v-card-title>

                <v-container>
                  <v-tabs v-model="tab">
                    <v-tab>画像のURLを貼り付け</v-tab>
                    <v-tab>画像のアップロード</v-tab>
                  </v-tabs>

                  <div v-if="tab == 0">
                    <v-text-field
                      v-model="uploadImageUrl"
                      class="mt-5"
                      placeholder="http://..."
                    ></v-text-field>
                  </div>

                  <div v-if="tab == 1">
                    <v-file-input
                      v-model="inputImage"
                      accept="image/*"
                      show-size
                      label="画像ファイルをアップロードしてください"
                      prepend-icon="mdi-image"
                      class="mt-5"
                      type="file"
                      @change="selectedFile"
                    />
                  </div>

                  <v-sheet class="pa-4" color="grey lighten-3">
                    <v-row class="my-5" align="center" justify="center">
                      <v-img contain max-height="200px" :src="uploadImageUrl" />
                    </v-row>
                  </v-sheet>
                </v-container>
              </v-card>
            </v-col>
          </v-row>
          <v-btn class="mt-5" block color="success" dark @click="post">{{
            $t('predict')
          }}</v-btn>
        </v-container>
      </v-card>

      <v-card class="my-10">
        <v-card-title class="primary white--text">
          {{ $t('検索結果') }}
        </v-card-title>
        <v-container>
          <div v-if="loadingFlag" class="text-center py-10">
            <v-progress-circular
              indeterminate
              color="primary"
            ></v-progress-circular>
          </div>
          <v-row>
            <v-col
              v-for="(type, index) in ['title', 'text', 'image']"
              :key="index"
              cols="12"
              sm="4"
            >
              <v-card
                v-if="results[type] && results[type].length > 0"
                outlined
                class="mb-5"
              >
                <v-card-title class="headline grey lighten-2">
                  {{ $t(type) }}
                </v-card-title>

                <v-container>
                  <v-simple-table>
                    <template v-slot:default>
                      <thead>
                        <tr>
                          <th class="text-left"></th>
                          <th class="text-left">タイプ</th>
                          <th class="text-left">確信度（0-1）</th>
                        </tr>
                      </thead>
                      <tbody>
                        <tr v-for="item in results[type]" :key="item.name">
                          <td>{{ item.name }}</td>
                          <td>{{ item.label }}</td>
                          <td>{{ item.prob }}</td>
                        </tr>
                      </tbody>
                    </template>
                  </v-simple-table>
                </v-container>
              </v-card>
            </v-col>
          </v-row>
        </v-container>
      </v-card>

      <v-expansion-panels class="my-10">
        <v-expansion-panel>
          <v-expansion-panel-header class="primary white--text">{{
            $t('reference')
          }}</v-expansion-panel-header>
          <v-expansion-panel-content>
            <v-container class="py-5">
              <ul>
                <li>
                  <a
                    target="_blank"
                    href="https://docs.google.com/spreadsheets/d/1ZXmqqhMofYFgy-f1VAhvYnUwpvcXPJJR7pY2ftjfBCw/edit?usp=sharing"
                    >学習曲線</a
                  >
                </li>
                <li>
                  <a
                    target="_blank"
                    href="https://www.kanzaki.com/works/2016/pub/image-annotator?u=https://raw.githubusercontent.com/nakamura196/testet/master/t2.json"
                    >教師データ</a
                  >
                </li>
              </ul>
            </v-container>
          </v-expansion-panel-content>
        </v-expansion-panel>
      </v-expansion-panels>
    </v-container>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'nuxt-property-decorator'
import axios from 'axios'

@Component
export default class Page extends Vue {
  head() {
    return {
      titleTemplate: null,
    }
  }

  title: string = ''
  text: string = ''
  results: any = {}

  inputImage: any = null

  uploadFile: any = null
  uploadImageUrl: any = ''

  loadingFlag: boolean = false

  tab: number = 0

  samples: any[] = [
    {
      title: 'Sword Guard (Tsuba)',
      text: `Classification: Tsuba
Credit Line: The Howard Mansfield Collection, Gift of Howard Mansfield, 1936
Culture: Japanese
Department: Arms and Armor
Dimensions: Diam. 3 3/16 in. (8.1 cm); thickness 3/16 in. (0.5 cm); Wt. 3.5 oz. (99.2 g)
Medium: Iron, copper
Object ID: 30060
Object Name: Sword guard (Tsuba)`,
      url:
        'https://images.metmuseum.org/CRDImages/aa/web-thumb/36.120.57_001mar2014.jpg',
    },
    {
      title: 'netsuke',
      text: `Description: 1936.21.0009
Description: Japan
Description: Liggande bock, som tittar bakåt över sin egen rygg. Enligt katalogen kamoshika, men det verkar inte stämma med hornens utformning. Hjortar betyder höst och vemod, särskilt kärlekssorg, men också symbol för långt liv.
MediaType: IMAGE
europeanaCollectionName: 91619_Ag_SE_SwedishNationalHeritage_smvk-em
id: 91619_SMVK_EM_objekt_1000932`,
      url:
        'https://api.europeana.eu/api/v2/thumbnail-by-url.json?uri=http%3A%2F%2Fcollections.smvk.se%2Fcarlotta-em%2Fweb%2Fimage%2Fzoom%2F1837468%2Fimage.jpg&type=IMAGE',
    },
    {
      title: 'Rumoi, Rumoi [Sub]prefecture. Hokkaido, Japan',
      text: `Description: Map — col. map. 56 x 53 cm.
Docmeta: 1st ed.-AMS 1. - Washington, 1945.`,
      url:
        'https://stacks.stanford.edu/image/iiif/bp444yn9111%2Fbp444yn9111_0001/full/!200,200/0/default.jpg',
    },
    {
      title: '石とプルナスの装飾が施されたカップ柿右衛門タイプ',
      text: `collection: Japanese Art
creditline: Bequest of Mrs. Severance A. Millikin
culture: Japan, Edo Period (1615-1868)
id: 154654
measurements: Diameter: 10.5 cm (4 1/8 in.); Overall: 5.8 cm (2 5/16 in.)
technique: porcelain with underglaze blue and overglaze enamel decoration, cursive "fuku" mark in underglaze blue
tombstone: Cup with Rock and Prunus Decoration: Kakiemon Type, 18th century. Japan, Edo Period (1615-1868). Porcelain with underglaze blue and overglaze enamel decoration, cursive "fuku" mark in underglaze blue; diameter: 10.5 cm (4 1/8 in.); overall: 5.8 cm (2 5/16 in.). The Cleveland Museum of Art, Bequest of Mrs. Severance A. Millikin 1989.256
type: Ceramic`,
      url: 'https://openaccess-cdn.clevelandart.org/1989.256/1989.256_web.jpg',
    },
    {
      title: 'マスク',
      text: `Description: 1901.24.0008
Description: Japan
Description: Mask till no-spel. (generalkatalogen)
MediaType: IMAGE
europeanaCollectionName: 91619_Ag_SE_SwedishNationalHeritage_smvk-em
id: 91619_SMVK_EM_objekt_1201810`,
      url:
        'https://api.europeana.eu/api/v2/thumbnail-by-url.json?uri=http%3A%2F%2Fcollections.smvk.se%2Fcarlotta-em%2Fweb%2Fimage%2Fzoom%2F1201812%2F1901.24.0008.jpg&type=IMAGE',
    },
  ]

  selectedFile(file: any) {
    if (file !== undefined && file !== null) {
      if (file.name.lastIndexOf('.') <= 0) {
        return
      }
      const fr = new FileReader()
      fr.readAsDataURL(file)
      fr.addEventListener('load', () => {
        this.uploadImageUrl = fr.result
      })
    } else {
      this.uploadImageUrl = ''
    }
  }

  post() {
    // 初期化
    this.results = {}

    this.loadingFlag = true

    const u = process.env.API_BASE_URL + '/ml'
    const title = this.title
    const text = this.text

    const inputImage = this.inputImage

    if (inputImage) {
      const formData = new FormData()
      formData.append('uploadFile', this.inputImage)
      formData.append('text', text)
      formData.append('title', title)
      formData.append('url', this.uploadImageUrl)

      const config: any = {
        headers: {
          'content-type': 'multipart/form-data',
        },
      }

      const self = this
      axios.post(u, formData, config).then((response) => {
        const all = response.data
        for (const key in all) {
          const result = all[key]
          const results = []
          for (let i = 0; i < result.length; i++) {
            const obj = result[i]
            const value = this.orgRound(Number(obj.prob), 2)
            if (value > 0) {
              results.push({
                name: '第' + (i + 1) + '候補',
                label: obj.label,
                prob: value,
              })
            }
          }
          all[key] = results
        }

        self.results = all
        self.loadingFlag = false
      })
    } else {
      const formData: any = {
        text,
        title,
        url: this.uploadImageUrl,
      }

      const self = this
      axios.get(u, { params: formData }).then((response) => {
        const all = response.data
        for (const key in all) {
          const result = all[key]
          const results = []
          for (let i = 0; i < result.length; i++) {
            const obj = result[i]
            const value = this.orgRound(Number(obj.prob), 2)
            if (value > 0) {
              results.push({
                name: '第' + (i + 1) + '候補',
                label: obj.label,
                prob: value,
              })
            }
          }
          all[key] = results
        }

        self.results = all
        self.loadingFlag = false
      })
    }
  }

  orgRound(value: number, n: number) {
    return Math.floor(value * Math.pow(10, n)) / Math.pow(10, n)
  }
}
</script>
