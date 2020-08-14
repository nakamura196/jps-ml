<template>
  <div>
    <v-container>
      <v-expansion-panels class="my-5">
        <v-expansion-panel>
          <v-expansion-panel-header>サンプル</v-expansion-panel-header>
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
                  @click="uploadImageUrl = obj.url"
                />
              </v-col>
            </v-row>
          </v-expansion-panel-content>
        </v-expansion-panel>
      </v-expansion-panels>
      <v-row>
        <v-col cols="12" sm="8"
          ><v-tabs v-model="tab">
            <v-tab>画像のURLを貼り付け</v-tab>
            <v-tab>画像のアップロード</v-tab>
          </v-tabs>

          <div v-if="tab == 0">
            <v-text-field
              v-model="uploadImageUrl"
              class="mt-5"
              placeholder="http://..."
            ></v-text-field>
            <v-btn class="mt-5" block color="primary" dark @click="postByUrl">{{
              $t('推測')
            }}</v-btn>
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

            <v-btn class="mt-5" block color="primary" dark @click="post">{{
              $t('推測')
            }}</v-btn>
          </div></v-col
        >
        <v-col cols="12" sm="4">
          <v-sheet class="pa-4" color="grey lighten-3">
            <v-row class="my-5" align="center" justify="center">
              <v-img contain max-height="200px" :src="uploadImageUrl" />
            </v-row>
          </v-sheet>
        </v-col>
      </v-row>

      <div v-if="loadingFlag" class="text-center py-10">
        <v-progress-circular
          indeterminate
          color="primary"
        ></v-progress-circular>
      </div>

      <v-simple-table v-if="results.length > 0" class="mt-10">
        <template v-slot:default>
          <thead>
            <tr>
              <th class="text-left"></th>
              <th class="text-left">タイプ</th>
              <th class="text-left">確信度（0-1）</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in results" :key="item.name">
              <td>{{ item.name }}</td>
              <td>{{ item.label }}</td>
              <td>{{ item.prob }}</td>
            </tr>
          </tbody>
        </template>
      </v-simple-table>

      <v-expansion-panels class="my-5 mt-10">
        <v-expansion-panel>
          <v-expansion-panel-header>参考資料</v-expansion-panel-header>
          <v-expansion-panel-content>
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

  text: string = ''
  results: any[] = []

  inputImage: any = null

  uploadFile: any = null
  uploadImageUrl: any = ''

  loadingFlag: boolean = false

  tab: number = 0

  samples: any[] = [
    {
      url:
        'https://images.metmuseum.org/CRDImages/aa/web-thumb/91.1.865_003AA2015.jpg',
    },
    {
      url:
        'https://api.europeana.eu/api/v2/thumbnail-by-url.json?uri=http%3A%2F%2Fcollections.smvk.se%2Fcarlotta-om%2Fweb%2Fimage%2Fzoom%2F382169%2Frepresentation&type=IMAGE',
    },
    {
      url:
        'https://api.europeana.eu/api/v2/thumbnail-by-url.json?uri=http%3A%2F%2Fcollections.smvk.se%2Fcarlotta-om%2Fweb%2Fimage%2Fzoom%2F375323%2Frepresentation&type=IMAGE',
    },
    {
      url:
        'https://stacks.stanford.edu/image/iiif/bp444yn9111%2Fbp444yn9111_0001/full/!200,200/0/default.jpg',
    },
    {
      url:
        'https://api.europeana.eu/api/v2/thumbnail-by-url.json?uri=http%3A%2F%2Fwww.kew.org%2Fherbcatimg%2F625710.jpg&type=IMAGE',
    },
    {
      url:
        'http://media.vam.ac.uk/media/thira/collection_images/2006AC/2006AC5478_jpg_ws.jpg',
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
    const u = 'http://localhost:8888/api/image'

    const formData = new FormData()
    formData.append('uploadFile', this.inputImage)
    const config: any = {
      headers: {
        'content-type': 'multipart/form-data',
      },
    }

    const self = this
    this.loadingFlag = true
    axios.post(u, formData, config).then((response) => {
      const result = response.data

      const results = []
      for (let i = 0; i < result.length; i++) {
        const obj = result[i]
        results.push({
          name: '第' + (i + 1) + '候補',
          label: obj.label,
          prob: obj.prob,
        })
      }
      self.results = results
      self.loadingFlag = false
    })
  }

  postByUrl() {
    const u = 'http://localhost:8888/api/imageByUrl'

    const params = new URLSearchParams()
    params.append('url', this.uploadImageUrl)

    const self = this
    this.loadingFlag = true
    axios.post(u, params).then((response) => {
      const result = response.data

      const results = []
      for (let i = 0; i < result.length; i++) {
        const obj = result[i]
        results.push({
          name: '第' + (i + 1) + '候補',
          label: obj.label,
          prob: obj.prob,
        })
      }
      self.results = results
      self.loadingFlag = false
    })
  }
}
</script>
