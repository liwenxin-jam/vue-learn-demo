<template>
  <div class="home">
    <child a="a" />
  </div>
</template>

<script lang="ts">
import { Component, Vue } from 'vue-property-decorator'
import Child from '@/components/child.vue' // @ is an alias to /src

@Component({
  components: {
    Child
  }
})
export default class Home extends Vue {}
</script>
