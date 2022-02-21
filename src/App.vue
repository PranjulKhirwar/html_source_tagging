<template>
  <div id="app">
    <RenderHTML :sourceHTML="sourceHTML" :offsetArr="offsetArr"></RenderHTML>
  </div>
</template>

<script>
import RenderHTML from './components/RenderHTML.vue'

export default {
  name: 'App',
  components: {
    RenderHTML
  },
  data(){
    return {
      sourceHTML: `<h2>Source Tagging for HTML</h2>
                  <p>This is a dummy application, with <b>dummy</b> text, but not so dummy logic!</p>
                  <div>
                    <p>I just lifted 100kg on the bench press, Now I'm gonna work on my squats</p>
                  </div>`,
      offsetArr: [{
        xpath: '/html/body/div/div/p',
        offsets: [[0,10], [2,8], [4,6],[20,24],[48,49]],
      }, {
        xpath: '/html/body/div/div/div/p',
        offsets: [[1,4],[9,16]],
      }]
    }
  }
}
</script>

<style>

</style>
