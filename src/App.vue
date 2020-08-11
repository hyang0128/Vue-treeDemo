<!--
 * @Author: Hyang
 * @Date: 2020-08-11 18:55:24
 * @LastEditors: Hyang
 * @LastEditTime: 2020-08-11 19:21:25
 * @Description: file content
-->
<template>
  <div id="app">
    <h2>Vue-TreeDemo</h2>
    <HelloWorld/>
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld'

export default {
  name: 'App',
  components: {
    HelloWorld
  }
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
