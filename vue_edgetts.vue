<template>
  <div>
    <div>
      <label for="text">输入文本：</label>
      <input type="text" id="text" v-model="text" />
    </div>
    <div>
      <label for="Gender">性别：</label>
      <select v-model="Gender" @change="updateSelected" title="gender">
        <option v-for="item in filteredList" :key="item.ShortName" :value="item.Gender">{{ item.Gender }}</option>
      </select>
    </div>
    <div>
      <label for="Locale">语言：</label>
      <select v-model="Locale" @change="updateSelected" title="locale">
        <option v-for="item in filteredList" :key="item.ShortName" :value="item.Locale">{{ item.Locale }}</option>
      </select>
    </div>
    <div>
      <label for="ShortName">选择音色：</label>
      <select v-model="ShortName" @change="updateSelected" title="gender">
        <option v-for="item in filteredList" :key="item.ShortName" :value="item.ShortName">{{ item.Locale }}</option>
      </select>
    </div>
    <div>
      <label for="volume">音量：</label>
      <input type="range" id="volume" v-model="volume" min="0" max="100" />
    </div>
    <div>
      <label for="rate">语速：</label>
      <input type="range" id="rate" v-model="rate" min="0" max="100" />
    </div>
    <button @click="speak">播放</button>
    <button @click="downloadAudio">下载音频</button>
    <button @click="downloadSubtitle">下载字幕</button>
  </div>
</template>

<script>
import { defineComponent, ref, reactive, computed } from 'vue';
import axios from 'axios';

export default defineComponent({
  name: 'Txt2Voice',

  setup() {
    const voice = ref("")
    const text = ref("")
    const volume = ref("")
    const rate = ref("")
    const Gender = ref("")
    const ShortName = ref("")
    const Locale = ref("")


    const selected = reactive({
      ShortName: null,
      Locale: null,
      Gender: null
    });
        

    async function getVoices() {
      const response = axios.get('http://127.0.0.1:5000/get_voice');
      let data = ((await response).data)

      return data;
    }

    const data = reactive(getVoices());

    const filteredList = computed(() => {

      if (selected.ShortName) {
        return data.list.filter(item => item.ShortName === selected.ShortName);
      } else if (selected.Locale) {
        return data.list.filter(item => item.Locale === selected.Locale);
      } else if (selected.Gender) {
        return data.list.filter(item => item.Gender === selected.Gender);
      } else {

        return data.list;
      }
    });

    
    selected.Gender = Gender.value; 
    selected.ShortName = ShortName.value; 
    selected.Locale = Locale.value;

    const updateSelected = () => {
      selected.ShortName = null;
      selected.Locale = null;
      selected.Gender = null;

      if (selected.Gender) {
        selected.Gender = this.Gender;
      }

      if (selected.Locale) {
        selected.Locale = this.Locale;
      }

      if (selected.ShortName) {
        selected.ShortName = this.ShortName;
      }
    };


    async function speak() {
      const url = 'http://127.0.0.1:5000/speak'
      let params = {
        text: text.value,
        voice: voice.value,
        volume: volume.value,
        rate: rate.value,
      };
      await axios.post(url, params);
    }

    return { voice, text, rate,volume,Gender,Locale,ShortName,selected, filteredList, updateSelected, speak, getVoices };
  },
  methods: {
    async downloadAudio() {
      const url = 'http://127.0.0.1:5000/download/audio';
      let params = {
        text: this.text,
        voice: this.voice,
        lang: this.lang,
        volume: this.volume,
        rate: this.rate,
      };
      await axios.post(url, params);
    },
    async downloadSubtitle() {
      const url = 'http://127.0.0.1:5000/download/subtitles';
      let params = {
        text: this.text,
        voice: this.voice,
        lang: this.lang,
        volume: this.volume,
        rate: this.rate,
      };
      await axios.post(url, params);
    },
  },
});
</script>

<style scoped></style>
