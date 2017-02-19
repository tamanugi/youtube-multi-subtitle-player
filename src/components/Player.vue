<template>
  <div class="hello">
      <div id="video-player"></div>
      <button v-on:click="loadYoutube">play!!</button>
      <div>{{ current }}</div>
      <div>{{ subtitle_ja }} </div>
      <div>{{ subtitle_en }} </div>
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
      this.player.loadVideoById('-sGiE10zNQM')
      this.player.stopVideo()
      this.player.on('ready', () => { this.update() })
      this.fetchLangSubtitle('ja')
      this.fetchLangSubtitle('en')
    },
    fetchLangSubtitle: function (lang) {
      let subtitleUrl = `https://www.youtube.com/api/timedtext?fmt=srv3&lang=${lang}&v=-sGiE10zNQM`
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
      let setCurrentTime = () => {
        if (this.player) {
          this.player.getCurrentTime().then(time => {
            time = Math.floor(time * 1000)
            this.current = time

            let subtitleJa = ''
            for (let i = 0; i < this.srt['ja'].length; i++) {
              let sub = this.srt.ja[i]
              let startTime = Number(sub.$.t)
              let endTime = startTime + Number(sub.$.d)
              if (startTime <= time && time <= endTime) {
                subtitleJa = sub._
                break
              }
            }
            let subtitleEn = ''
            for (let i = 0; i < this.srt['en'].length; i++) {
              let sub = this.srt.en[i]
              let startTime = Number(sub.$.t)
              let endTime = startTime + Number(sub.$.d)
              if (startTime <= time && time <= endTime) {
                subtitleEn = sub._
                break
              }
            }
            this.subtitle_ja = subtitleJa
            this.subtitle_en = subtitleEn
          })
        }
      }
      setCurrentTime()
      setTimeout(this.update, 50)
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
