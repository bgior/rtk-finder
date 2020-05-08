<!-- © Copyright 2020 Bruno Giorello. Released under GNU AGPLv3, see 'LICENSE.md'. -->
<template>
  <div>
    <input type="text" id="txtQuery" v-focus v-model="query" placeholder="Type a keyword or component..." @keydown="evt => handleKeydown(evt)" aria-label="Search"/>
    <div v-if="queryPresent">
      <div v-if="searchResults.length > 0" class="row">
        <div v-for="(kanji, idx) in limitedSearchResults" class="col-12 col-sm-6 mt-2" :key="kanji.char" @click="copyKanji(kanji)">
          <div :class="{'card result': true, selected: idx == selectedResultIndex}">
            <div class="character" v-html="highlightCharacter(kanji)"></div>
            <div class="text">
              <div class="keyword" v-html="highlightKeyword(kanji)"></div>
              <div class="components" v-html="highlightComponents(kanji)"></div>
            </div>
            <transition name="fade">
              <span v-if="kanji.showCopyMessage" class="copy-notice">COPIED</span>
            </transition>
          </div>
        </div>
      </div>
      <div v-else class="notice mt-4">No results found</div>
    </div>
    <div v-else id="divPlaceholder">
      検索
    </div>
    <div v-if="searchResults.length > limitedSearchResults.length" class="notice mt-4">
      {{ searchResults.length - limitedSearchResults.length }} more kanji found.<br/>
      <template v-if="max_results == 10"><span class="clickable-text" @click="allowMoreResults()">Show more</span></template>
      <template v-else><span class="clickable-text" @click="allowFewerResults()">Show fewer</span></template>
    </div>
  </div>
</template>

<script>
import Vue from 'vue';
import Kanji from '@/assets/data/kanji.json'
const DEFAULT_MAX_RESULTS = 10; // Maximum number of results to show at once as default
const EXTENDED_MAX_RESULTS = 200; // Maximum number of results to show at once after the user clicks 'Show more'

export default {
  name: 'Search',
  data: () => { return {
    query: "",
    selectedResultIndex: -1,
    max_results: DEFAULT_MAX_RESULTS
  }},
  computed: {
    // Return true if there's a valid query entered
    queryPresent() {
      return this.sanitizedQuery.length > 0;
    },
    // Return the query without most special characters, which are useless and would have to be escaped on regexes
    sanitizedQuery() {
      return this.query.replace(/[^a-zA-Z0-9 '\u4E00-\u9FBF]/g, '');
    },
    // A regex to match any and all of the query items, which will be used to highlight them
    regexToHighlight() {
      return new RegExp(`(${this.queryItems.map(it => it.text).join('|')})`, 'gi');
    },
    // Return the list of search results, each being a kanji object
    searchResults() {
      if (this.queryPresent) {
        const queryItems = this.queryItems;
        return Kanji
          .filter(k => {
            for (let item of queryItems) {
              if (k.dword.includes(item.text) || (k.alt && k.alt.includes(item.text)) || (k.components && item.regexToTest.test(k.components)) || item.text.includes(k.char)) {
                k.relevance = this.relevanceToQuery(k);
                return true;
              }
            }
            return false;
          })
          .sort((k1, k2) => k2.relevance - k1.relevance);
      } else {
        return [];
      }
    },
    limitedSearchResults() {
      return this.searchResults.slice(0, this.max_results);
    },
    // Grab the query string and return a list of items to be searched for individually
    queryItems() {
      return this.sanitizedQuery.split(' ').filter(i => i.length > 0).map(text => {
        return {
          text: text.toLowerCase(),
          // Create the regexes here so that they can be reused
          regexToTest: new RegExp(`${text}`, 'i'),
          regexToTestAlone: new RegExp(`(?:^|[^a-zA-Z])${text}[^a-zA-Z]`, 'i')
        }
      })
    }
  },
  methods: {
    highlightCharacter(kanji) {
      for (let item of this.queryItems) {
        if (item.text.includes(kanji.char)) {
          return `<span class="highlight">${kanji.char}</span>`;
        }
      }
      return kanji.char;
    },
    highlightKeyword(kanji) {
      return kanji.word.replace(this.regexToHighlight, '<span class="highlight">$1</span>');
    },
    highlightComponents(kanji) {
      if (kanji.components) {
        return kanji.components.replace(this.regexToHighlight, '<span class="highlight">$1</span>');
      } else {
        return null;
      }
    },
    // Assign a score to each kanji based on how relevant it is to the user's query
    relevanceToQuery(kanji) {
      let score = 0;
      for (let item of this.queryItems) {
        if (item.text.includes(kanji.char)) {
          score += 200;
        } else if (kanji.dword.includes(item.text)) {
          if (kanji.dword == item.text) {
            score += 100;
          } else if (kanji.dword.startsWith(item.text)) {
            score += 90;
          } else {
            score += 80;
          }
        } else if (kanji.alt && kanji.alt.includes(item.text)) {
          if (kanji.alt == item.text) {
            score += 99;
          } else if (kanji.alt.startsWith(item.text)) {
            score += 89;
          } else {
            score += 79;
          }
        } else if (item.regexToTest.test(kanji.components)) {
          if (item.regexToTestAlone.test(kanji.components)) {
            score += 70;
          } else {
            score += 69;
          }
        }
      }
      return score;
    },
    // Given a kanji object, copy the character to clipboard
    copyKanji(kanji) {
      navigator.clipboard.writeText(kanji.char);
      Vue.set(kanji, 'showCopyMessage', true);
      setTimeout(() => {
        Vue.set(kanji, 'showCopyMessage', false);
        this.$forceUpdate();
      }, 2500);
      this.$forceUpdate();
    },
    // Handle special keyboard actions from the query input
    handleKeydown(evt) {
      switch(evt.keyCode) {
        case 13: // Enter key
          if (this.limitedSearchResults[this.selectedResultIndex]) {
            this.copyKanji(this.limitedSearchResults[this.selectedResultIndex]);
          } else if (this.limitedSearchResults.length > 0) {
            this.copyKanji(this.limitedSearchResults[0]);
          }
          break;
        case 38: // Up arrow
          if (this.selectedResultIndex > -1) {
            this.selectedResultIndex--;
          }
          break;
        case 40: // Down arrow
          if (this.selectedResultIndex < this.limitedSearchResults.length - 1) {
            this.selectedResultIndex++;
          }
          break;
      }
    },
    allowMoreResults() {
      this.max_results = EXTENDED_MAX_RESULTS;
    },
    allowFewerResults() {
      this.max_results = DEFAULT_MAX_RESULTS;
    }
  },
  created() {
    for (let kanji of Kanji) {
      kanji.dword = kanji.word.toLowerCase(); // Add the downcased keyword as an extra pre-computed attribute for more performant searches
      if (kanji.alt) {
        kanji.alt = kanji.alt.toLowerCase(); // We don't need to create a lower cased version of this attribute because we don't show it anyways
      }
    }
  },
  watch: {
    limitedSearchResults() {
      // Whenever the results change, unselect any selected results, because the index probably no longer corresponds to the result it used to
      this.selectedResultIndex = -1;
    }
  }
}
</script>

<style>
  #txtQuery {
    width: 100%;
    font-size: 20px;
    padding: 10px;
    box-shadow: 1px 1px 3px #aaa;
    border: 1px solid #bbb;
    border-radius: 4px;
  }
  #txtQuery:focus {
    border-color: #ffa500;
  }
  #divPlaceholder {
    text-align: center;
    font-size: 120px;
    color: #e8e8e8;
    padding-top: 40px;
    text-shadow: 1px 1px 1px #ccc;
    user-select: none;
  }
  .result {
    display: flex;
    cursor: pointer;
    transition: box-shadow 0.3s;
    height: 90px;
  }
  .result.selected {
    background-color: #f09b00;
    color: white;
    text-shadow: 1px 1px 2px #222;
  }
  .result.selected .highlight {
    color: white;
  }
  .result.selected .components {
    color: #eed;
  }
  .result:hover {
    box-shadow: 1px 1px 5px #444;
  }
  .character {
    margin-top: 5px;
    font-family: 'KosugiMaru';
    font-size: 56px;
    width: 75px;
    height: 75px;
    text-align: center;
    line-height: 75px;
    flex-shrink: 0;
  }
  .text {
    display: flex;
    flex-direction: column;
    justify-content: center;
    overflow: hidden;
  }
  .keyword {
    font-size: 26px;
    font-weight: bold;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
  }
  .highlight {
    color: #f09b00;
    font-weight: bold;
  }
  .components {
    font-size: 12px;
    color: #666;
    max-height: 40px;
  }
  .notice {
    color: grey;
    font-size: 14px;
    text-align: center;
  }
  .copy-notice {
    opacity: 0.9;
    position: absolute;
    bottom: -6px;
    left: 23px;
    background-color: #549308;
    color: white;
    border-radius: 4px;
    padding: 4px 4px 0px 4px;
    font-weight: bold;
    font-size: 12px;
    border: 2px solid #45750a;
  }
</style>
