<template>
  <div id="app">
<!--    <MessageUi />-->
<!--    <MessageUi2 />-->
    <MessageUi3 />
  </div>
</template>

<script>
import HelloWorld from './components/HelloWorld'
import MessageUi from './components/MessageUI'
import MessageUi2 from './components/MessageUI2'
import MessageUi3 from './components/MessageUI3'

export default {
  name: 'App',
  components: {
    HelloWorld,
    MessageUi,
    MessageUi2,
    MessageUi3
  }
}
</script>

<style>

</style>
