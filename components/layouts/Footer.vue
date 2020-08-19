<template>
  <v-footer :dark="true" class="mt-5">
    <v-container>
      <p class="text-center my-5" style="color: white;">
        ジャパンサーチ利活用スキーマを用いた機械分類サンプル
      </p>
    </v-container>
  </v-footer>
</template>

<script lang="ts">
import { Vue, Component } from 'nuxt-property-decorator'

@Component
export default class Footer extends Vue {}
</script>
