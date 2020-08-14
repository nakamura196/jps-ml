<template>
  <div>
    <v-container>
      <v-textarea
        v-model="text"
        class="my-5"
        filled
        label="Text"
        auto-grow
      ></v-textarea>

      <v-btn block color="primary" dark @click="post">{{ $t('推測') }}</v-btn>

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

  post() {
    const text = this.text

    const u = 'http://localhost:8888/api'

    const params = new URLSearchParams()
    params.append('text', text)

    const self = this
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
    })
  }
}
</script>
