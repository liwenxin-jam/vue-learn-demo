<template>
  <div>child</div>
</template>

<script lang="ts">
import { Component, Vue, Prop } from 'vue-property-decorator'
// import Child from '@/components/child.vue' // @ is an alias to /src

@Component({
  // components: {
  //   Child
  // }
  inheritAttrs: false
})
export default class Child extends Vue {
  // @Prop() a?: string
}
</script>
