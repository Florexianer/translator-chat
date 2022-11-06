<template>
  <div>
    <div style="display: flex; justify-content: space-around">
      <div></div>
      <div style="display: flex; justify-content: space-around; width: 30%">
        <div style="text-align: center">
          From <br>
          <select name="from" id="from" v-model="source" @change="this.history = []">
            <option :value="language.code" v-for="(language, i) in languages" :key="i">{{ language.name }}</option>
          </select>
        </div>
        <div style="text-align: center">
          To <br>
          <select name="to" id="to" v-model="target" @change="this.history = []">
            <option :value="language.code" v-for="(language, i) in languages" :key="i">{{ language.name }}</option>
          </select>
        </div>
      </div>
      <div></div>
    </div>
    <div class="container">
      <div class="imessage">
        <div v-for="(item, i) in history" :class="item.side" :key="i">
          <div style="width: fit-content">{{item.input}}</div>
          <hr>
          <div style="width: fit-content">
            {{ item.output }}
            <button @click="talk(item.output)" style="border: none; background-color: transparent" class="listenButton">
            <span class="material-symbols-outlined play">
              volume_up
            </span>
            </button>
          </div>
        </div>

        <span class="material-symbols-outlined" v-if="side === 'l' && show" style="position:relative; left: 0; bottom: 0; display: block; width: 30px;!important;">
          arrow_forward_ios
        </span>
        <span class="material-symbols-outlined" v-if="side === 'r' && show" style="position:relative; left: calc(100% - 15px); bottom: 0; display: block; width: 30px!important;">
        arrow_back_ios
      </span>
        <div id="bottom" style="position:relative; left: 20%; bottom: 0; display: block; min-height: 20px; max-width: 20px">
        </div>
      </div>
      <div class="comment">
        <button @click="listen()" style="min-width: 13%; min-height: 40px" class="listenButton">
          <span class="material-symbols-outlined nofill" id="nofitem">
            mic
          </span>
        </button>

        <button @click="side = side === 'l' ? 'r' : 'l'" style="min-width: 13%; min-height: 40px" class="listenButton">
          <span class="material-symbols-outlined">
            autorenew
          </span>
        </button>

        <input type="text" v-model="input" style="min-width: 59%; border-radius: 60px; min-height: 35px; padding-left: 5px;">

        <button @click="translate" style="min-width: 13%; min-height: 40px" class="listenButton">
          <span class="material-symbols-outlined">
            send
          </span>
        </button>

      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  name: "MainComponent",
  data() {
    return {
      languages: {},
      source : 'en',
      target : 'de',
      input: 'How are you doing broski',
      side: 'l',
      history: [
      ],
      show: null
    }
  },
  async created() {
    this.languages = (await axios.get('https://libretranslate.de/languages')).data
    this.showV()
  },
  methods: {
    showV () {
      this.show = true
      setTimeout(this.hide,1200)
    },

    hide() {
      this.show = false
      setTimeout(this.showV,1200)
    },

    async translate() {
      const [curLang,curTarg] = this.side === 'l' ? [this.source, this.target] : [this.target, this.source]

      const output = (await axios.post('https://libretranslate.de/translate', {
        q: this.input,
        source: curLang,
        target: curTarg,
        format: 'text',
      })).data.translatedText
      this.history.push({input: this.input, output: output, side: this.side, lang: curTarg})

      setTimeout(this.scrollDown, 10)

      this.side = this.side === 'l' ? 'r' : 'l'
      this.input = ''
    },

    scrollDown() {
      let element = document.getElementById('bottom')
      element.scrollIntoView({ behavior: 'smooth', block: 'end' })
    },

    talk(text) {
      let speech = new SpeechSynthesisUtterance();
      speech.lang = 'ja-JP'
      speech.text = text
      window.speechSynthesis.speak(speech)
    },

    listen () {
      const SpeechRecognition =  window.SpeechRecognition || window.webkitSpeechRecognition;
      let recognition = new SpeechRecognition();
      const item = document.querySelector('#nofitem')

      recognition.onstart = () => {
        item.classList.remove('nofill')
      }

      recognition.onspeechend = () => {
        item.classList.add('nofill')
        recognition.stop();
      }

      recognition.onresult = (result) => {
        let vocalInput = result.results[0][0].transcript;
        this.input += vocalInput;
      }

      recognition.onerror = (event) => {
        item.classList.add('nofill')
        console.error('Error: ' + event.error + '\n Speech recognition is not working in brave');
      }

      recognition.start()
    },
  },
}
</script>

<style scoped>

.play {
  color: black;
}

.material-symbols-outlined.nofill {
  font-variation-settings:
      'FILL' 0
}

.material-symbols-outlined {
  font-variation-settings:
      'FILL' 1,
      'wght' 600,
      'GRAD' 0,
      'opsz' 48
}

.listenButton:hover {
  cursor: pointer;
}

.listenButton {
  font-size: 18px;
  border-radius: 60px;
  color: #248bf5;
  background-color: white;
}

.imessage {
  height: 1000px;
  background-color: #fff;
  border: 1px solid #e5e5ea;
  border-radius: 0.25rem;
  display: flex;
  flex-direction: column;
  font-family: "SanFrancisco";
  font-size: 1.25rem;
  margin: 0 auto 1rem;
  max-width: 600px;
  padding: 0.5rem 1.5rem;
  overflow-y:scroll;
}

.imessage div {
  border-radius: 1.15rem;
  line-height: 1.25;
  max-width: 75%;
  padding: 0.5rem .875rem;
  position: relative;
  word-wrap: break-word;
}

.imessage div::before,
.imessage div::after {
  bottom: -0.1rem;
  content: "";
  height: 1rem;
  position: absolute;
}

div.r {
  align-self: flex-end;
  background-color: #248bf5;
  color: #fff;
}

div.r::before {
  border-bottom-left-radius: 0.8rem 0.7rem;
  border-right: 1rem solid #248bf5;
  right: -0.35rem;
  transform: translate(0, -0.1rem);
}

div.r::after {
  background-color: #fff;
  border-bottom-left-radius: 0.5rem;
  right: -40px;
  transform:translate(-30px, -2px);
  width: 10px;
}

div.l, div.r {
  margin: 0.5rem 0;
  width: fit-content;
}

div.r ~ div.r {
  margin: 0.25rem 0 0;
}

div.r ~ div.r:not(:last-child) {
  margin: 0.25rem 0 0;
}

div.r ~ div.r:last-child {
  margin-bottom: 0.5rem;
}

div.l {
  align-items: flex-start;
  background-color: #e5e5ea;
  color: #000;
}

div.l:before {
  border-bottom-right-radius: 0.8rem 0.7rem;
  border-left: 1rem solid #e5e5ea;
  left: -0.35rem;
  transform: translate(0, -0.1rem);
}

div.l::after {
  background-color: #fff;
  border-bottom-right-radius: 0.5rem;
  left: 20px;
  transform: translate(-30px, -2px);
  width: 10px;
}

.no-tail::before {
  display: none;
}

.margin-b_none {
  margin-bottom: 0 !important;
}

.margin-b_one {
  margin-bottom: 1rem !important;
}

.margin-t_one {
  margin-top: 1rem !important;
}














/* general styling */
@font-face {
  font-family: "SanFrancisco";
  src:
      url("https://cdn.rawgit.com/AllThingsSmitty/fonts/25983b71/SanFrancisco/sanfranciscodisplay-regular-webfont.woff2") format("woff2"),
      url("https://cdn.rawgit.com/AllThingsSmitty/fonts/25983b71/SanFrancisco/sanfranciscodisplay-regular-webfont.woff") format("woff");
}

* {
  font-family: -apple-system,
  BlinkMacSystemFont,
  "Segoe UI",
  Roboto,
  Oxygen-Sans,
  Ubuntu,
  Cantarell,
  "Helvetica Neue",
  sans-serif;
  font-weight: normal;
  margin: 0;
  font-size: 1.25rem;
}

.container {
  margin: 0 auto;
  max-width: 600px;
  padding: 1rem;
}

h1 {
  font-weight: normal;
  margin-bottom: 0.5rem;
}

h2 {
  border-bottom: 1px solid #e5e5ea;
  color: #666;
  font-weight: normal;
  margin-top: 0;
  padding-bottom: 1.5rem;
}

.comment {
  color: #222;
  font-size: 1.25rem;
  line-height: 1.5;
  margin-bottom: 1.25rem;
  max-width: 100%;
  padding: 0;
}
</style>