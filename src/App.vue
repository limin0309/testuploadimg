<template>
  <div id="app">
    <img alt="Vue logo" src="./assets/images/logo.png">
     <HelloWorld v-modal="imgList" @saveList="saveList" :value="imgList">
       <span style="color:blue">上传图片</span>
     </HelloWorld>
  </div>
</template>
<script>
// 这个helloworld是a.vue
import HelloWorld from './views/HelloWorld.vue'
export default {
  name: 'App',
  components: {
    HelloWorld,
  },
    data() {
    return {
      imgList:['https://wpimg.wallstcn.com/577965b9-bb9e-4e02-9f0c-095b41417191']
    }
  },
  methods: {
    saveList(vaule){
      console.log('提交图片',vaule)
    }
  }


}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
