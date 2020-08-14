<template>
  <v-footer :dark="true" class="mt-5">
    <v-container>
      <p class="text-center my-5" style="color: white;">
        中村覚（国立国会図書館 非常勤調査員 業務成果）
      </p>
    </v-container>
  </v-footer>
</template>

<script lang="ts">
import { Vue, Component } from 'nuxt-property-decorator'

@Component
export default class Footer extends Vue {}
</script>
