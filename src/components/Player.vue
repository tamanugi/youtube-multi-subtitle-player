<template>
  <div class="player">
    <div id="video-player"></div>
    <div>{{ subtitle_ja }}</div>
    <div>{{ subtitle_en }}</div>
    <div>
      <input type="text" v-model="videoid"><button v-on:click="loadYoutube">play!!</button>
    </div>
    <div>{{ current }}</div>
  </div>
</template>

<script>
import YouTubePlayer from 'youtube-player'
import http from 'http'
import {parseString} from 'xml2js'

export default {
  name: 'player',
  data () {
    return {
      videoid: '-sGiE10zNQM',
      current: 0,
      player: {},
      srt: {},
      subtitle_ja: '',
      subtitle_en: ''
    }
  },
  methods: {
    loadYoutube: function () {
      this.player = YouTubePlayer('video-player')
      this.player.loadVideoById(this.videoid)
      this.player.stopVideo()
      this.player.on('ready', () => { this.update() })
      this.fetchLangSubtitle('ja')
      this.fetchLangSubtitle('en')
    },
    fetchLangSubtitle: function (lang) {
      let subtitleUrl = `https://www.youtube.com/api/timedtext?fmt=srv3&lang=${lang}&v=${this.videoid}`
      console.log(subtitleUrl)
      http.get(subtitleUrl, (res) => {
        let body = ''
        res.setEncoding('utf8')

        res.on('data', (chunk) => {
          body += chunk
        })

        res.on('end', (res) => {
          parseString(body, (err, result) => {
            if (!err) console.dir(result)
            this.srt[lang] = result.timedtext.body[0].p
          })
        })
      }).on('error', (e) => {
        console.log(e.message) // エラー時
      })
    },
    update: function () {
      if (this.player) {
        this.player.getCurrentTime().then(time => {
          time = Math.floor(time * 1000)
          this.current = time

          this.subtitle_ja = this.getSubtitle('ja')
          this.subtitle_en = this.getSubtitle('en')
        })
      }
      setTimeout(this.update, 50)
    },
    getSubtitle: function (lang) {
      let subtitle = ''
      for (let i = 0; i < this.srt[lang].length; i++) {
        let sub = this.srt[lang][i]
        let startTime = Number(sub.$.t)
        let endTime = startTime + Number(sub.$.d)
        if (startTime <= this.current && this.current <= endTime) {
          subtitle = sub._
          break
        }
      }
      return subtitle
    }
  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
