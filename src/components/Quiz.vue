<!-- © Copyright 2020 Bruno Giorello. Released under GNU AGPLv3, see 'LICENSE.md'. -->
<template>
  <div>
    <hr/>
    <div class="d-flex justify-content-between">
      <span class="pt-1" style="flex-shrink: 0">
        Quiz type:
        <label class="ml-2"><input type="radio" v-model="kanjiToKeyword" :value="true" @change="regenerateQuiz()"/> Recall keyword</label>
        <label class="ml-2"><input type="radio" v-model="kanjiToKeyword" :value="false" @change="regenerateQuiz()"/> Recall kanji</label>
      </span>
      <button id="btnRegen" @click="regenerateQuiz()" style="float: right">
        <img src="@/assets/img/refresh.min.svg"/>
      </button>
    </div>
    <hr/>
    <div class="row">
      <div v-for="kanji in randomSelection" class="col-12 col-md-6 mt-2" :key="kanji.char" @mouseenter="reveal(kanji)" @mouseleave="hide(kanji)">
        <div class="card result">
          <div :class="{character: true, 'pt-2': !kanjiToKeyword && !kanji.revealed}">{{ kanjiToKeyword || kanji.revealed ? kanji.char : '?' }}</div>
          <div class="text">
            <div class="keyword">{{ !kanjiToKeyword || kanji.revealed ? completeName(kanji) : '?' }}</div>
            <div class="components">{{ kanji.revealed ? kanji.components : '???? ???? ????' }}</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Vue from 'vue'
import Kanji from '@/assets/data/kanji.json'
const QUIZ_SIZE = 10;

export default {
  name: 'Quiz',
  data: () => { return {
    kanjiToKeyword: true,
    randomSelection: null
  }},
  methods: {
    regenerateQuiz() {
      this.randomSelection = JSON.parse(JSON.stringify(Kanji.sort(() => 0.5 - Math.random()).slice(0, QUIZ_SIZE)));
    },
    reveal(kanji) {
      Vue.set(kanji, 'revealed', true);
    },
    hide(kanji) {
      kanji.revealed = false;
    },
    completeName(kanji) {
      return kanji.alt ? `${kanji.word}/${kanji.alt}` : kanji.word;
    }
  },
  created() {
    this.regenerateQuiz();
  }
}
</script>

<style>
  #btnRegen {
    background-color: #ec9900;
    box-shadow: 1px 1px 2px #777;
    cursor: pointer;
    float: right;
    padding: 4px;
    height: 30px;
    border: 1px solid #a66b00;
    border-radius: 50%;
    transition: box-shadow 0.4s, filter 0.4s;
    user-select: none;
    outline: 0;
  }
  #btnRegen > img {
    width: 20px;
    filter: drop-shadow(1px 1px 2px #555);
  }
  #btnRegen:hover {
    filter: brightness(1.1);
  }
</style>
