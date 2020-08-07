<!-- © Copyright 2020 Bruno Giorello. Released under GNU AGPLv3, see 'LICENSE.md'. -->
<template>
  <div class="card px-4 mx-2">
    <h1>RTKFinder</h1>
    <p>
      This tool allows Japanese learners using James Heisig's RTK (<a href="https://en.wikipedia.org/wiki/Remembering_the_Kanji_and_Remembering_the_Hanzi">Remembering the Kanji</a>) method to easily look up kanji through its keyword or vice versa.
    </p>
    <p>This project is open-source and available on <a href="https://github.com/bgior/rtk-finder">GitHub</a>.</p>
    <h2>Tips</h2>
    <ul>
      <li>Use the up and down arrows to move between the search results</li>
      <li>Use the Enter key to copy the selected kanji (or the first result, if none selected) to the clipboard.</li>
    </ul>

    <h2>What happened to the primitives?</h2>
    <p>The app used to include primitives and the ability to search by them, but unfortunately this feature has been removed to avoid copyright issues.</p>
  </div>
</template>

<script>
export default {
  name: 'About'
}
</script>