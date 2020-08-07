<!-- © Copyright 2020 Bruno Giorello. Released under GNU AGPLv3, see 'LICENSE.md'. -->
<template>
  <div id="app">
    <div class="container-fluid">
      <div class="tab-heads">
        <div :class="{ 'tab-head': true, selected: selectedTabIndex == 1 }" @click="selectedTabIndex = 1">Quiz</div>
        <div :class="{ 'tab-head': true, selected: selectedTabIndex == 2 }" @click="selectedTabIndex = 2">About</div>
      </div>
      <div class="row justify-content-center">
        <div class="col-12 col-md-10 col-lg-8 col-xl-7">
          <div id="divLogo" class="mb-2"><img src="./assets/img/logo.png" @click="selectedTabIndex = 0" alt="Logo"/></div>
          <transition name="fade" mode="out-in">
            <Search v-if="selectedTabIndex == 0"/>
            <Quiz v-if="selectedTabIndex == 1"/>
            <About v-if="selectedTabIndex == 2"/>
          </transition>
        </div>
      </div>
    </div>
    <span style="font-family: 'KosugiMaru'; visibility: hidden">RTKF</span> <!-- Firefox doesn't seem to prefetch the Japanese font despite the directive, waiting until a kanji is shown instead, so this hidden span does the trick -->
  </div>
</template>

<script>
import Vue from 'vue';
import Search from './components/Search.vue'
import Quiz from './components/Quiz.vue'
import About from './components/About.vue'

import '@/assets/css/bootstrap-grid.min.css'

export default {
  name: 'App',
  components: {
    Search,
    Quiz,
    About
  },
  data: () => {
    return {
      selectedTabIndex: 0
    }
  }
}

Vue.directive('focus', {
  inserted: function (el) {
    el.focus()
  }
})
</script>

<style>
@font-face {
  font-family: 'Ubuntu';
  font-style: normal;
  font-weight: 400;
  font-display: swap;
  src: local('Ubuntu Regular'), local('Ubuntu-Regular'), url('./assets/fonts/ubuntu.woff2') format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+2000-206F, U+2074, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
@font-face {
  font-family: 'KosugiMaru';
  font-style: normal;
  font-weight: 400;
  font-display: swap;
  src: url('./assets/fonts/KosugiMaru-Regular.woff2') format('woff2');
}
body {
  background-color: #eee;
}
#app {
  font-family: 'Ubuntu', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #2c3e50;
}
.tab-heads {
  display: flex;
  justify-content: space-between;
  align-items: center;
}
.tab-head {
  padding: 5px;
  border-radius: 4px;
  font-weight: bold;
  color: #777;
  cursor: pointer;
}
.tab-heads > div.selected {
  color: #ffa500;
}
.card {
  border-radius: 4px;
  background-color: white;
  box-shadow: 1px 1px 3px #aaa;
  padding: 4px;
}
a, .clickable-text {
  cursor: pointer;
  color: #cc8400;
}
hr {
  border: 1px solid #ccc;
}
h2 {
  font-size: 22px;
}
ul {
  padding-left: 30px;
}
li {
  margin-bottom: 5px;
}
input[type='radio']:after {
    width: 15px;
    height: 15px;
    border-radius: 15px;
    top: -2px;
    left: -1px;
    position: relative;
    background-color: white;
    content: '';
    display: inline-block;
    visibility: visible;
    border: 2px solid white;
    box-shadow: 0px 0px 2px #555;
}
input[type='radio']:checked:after {
    width: 15px;
    height: 15px;
    border-radius: 15px;
    top: -2px;
    left: -1px;
    position: relative;
    background-color: #ffa500;
    content: '';
    display: inline-block;
    visibility: visible;
    border: 2px solid white;
}
.fade-enter-active, .fade-leave-active {
  transition: opacity .2s;
}
.fade-enter, .fade-leave-to {
  opacity: 0;
}
#divLogo {
  text-align: center;
}
#divLogo > img {
  width: 60px;
  filter: drop-shadow(1px 1px 2px #888);
  cursor: pointer;
}
@media (max-width: 800px) {
  #divLogo > img {
    width: 40px;
    margin-top: -20px;
  }
}
</style>
