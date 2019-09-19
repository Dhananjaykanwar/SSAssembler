<template>
  <v-app class="mainPage">
    <v-app-bar app>
      <v-toolbar-title class="headline text-uppercase">
        <span>DK</span>
        <span class="font-weight-light">SG</span><span>Assembler</span>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn color='blue' @click="example">
        <span>Run</span>
      </v-btn>
      <v-btn ml-5 color='red' @click="reload">
        <span>Reset</span>
      </v-btn>
    </v-app-bar>

    <v-content>
      <HelloWorld ref="HelloWorld"/>
    </v-content>
  </v-app>
</template>

<script>
import HelloWorld from './components/HelloWorld';
export default {
  name: 'App',
  components: {
    HelloWorld,
  },
  methods:{
    example(){
      this.$refs.HelloWorld.readSeparate();
    },
    reload(){
      window.location.reload();
    }
  },
  data: () => ({
    //
  }),
};
</script>
<style>
</style>
