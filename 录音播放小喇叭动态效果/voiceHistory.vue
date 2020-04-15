<template>
    <div class="voiceMsg">
      <div class="voiceBox" @click="audioPlay">
        <div class="wifi-symbol">
            <div class="wifi-circle wifi-first"></div>
            <div class="wifi-circle pause-second" :class="{'wifi-second': !ended}"></div>
            <div class="wifi-circle pause-third" :class="{'wifi-third': !ended}"></div>
        </div>{{voiceObj.voiceInfo.time}}"
      </div>
      <p class="voiceCon">{{voiceObj.voiceInfo.content}}</p>
    </div>
</template>
<script>
export default {
  name: 'voiceHistory',
  props: {
    voiceObj: {
      type: Object
    },
    thisVoice: {
      type: String
    }
  },
  data () {
    return {
      audioPlayer: new Audio(),
      ended: true,
      url: this.voiceObj.voiceInfo.sourcePath,
    }
  },
  beforeDestroy () { // 页面关闭停止播放语音
    this.audioPlayer.pause()
  },
  watch: {
    thisVoice (to, from) {
      if (to !== this.url) {
        this.playEnd()
      }
    }
  },
  methods: {
    audioPlay () {
      this.$emit('voiceMth', this.url)
      setTimeout(() => {
        this.playMth()
      }, 10)
    },
    playMth () {
      // if (this.audioPlayer.src !=)
      console.log('this.thisVoice', this.thisVoice)
      if (this.audioPlayer.paused) {
        this.playStart()
      } else {
        this.playEnd()
      }
      let voiceTimer = setInterval(() => {
        console.log(this.audioPlayer.networkState, this.audioPlayer.readyState, this.audioPlayer)
        if (this.audioPlayer.ended || this.audioPlayer.paused) {
          clearInterval(voiceTimer)
          this.ended = true
        }
        // if (this.audioPlayer.networkState == 3 && this.audioPlayer.readyState <= 1) { // 不支持播放
        //   clearInterval(voiceTimer)
        //   this.$message({
        //     type: 'warning',
        //     message: '语音播放异常'
        //   })
        //   this.ended = true
        // }
      }, 300)
    },
    playEnd () {
      this.audioPlayer.src = ''
      this.audioPlayer.pause()
      this.ended = true
    },
    playStart () {
      // this.audioPlayer.currentTime = 0
      this.audioPlayer.src = this.url
      this.audioPlayer.play()
      this.ended = false
    }
  },
}
</script>
<style scoped>
  .voiceMsg {
    position: relative;
    width: 100px;
  }
  .voiceBox {
    word-wrap:break-word;
    word-break:break-all;
    text-indent:35px;
    background:#b3e866;
    border:#ccc;
    cursor:pointer;
    border-radius:3px;
    width:100px;
    line-height:25px;
    height:25px;
    position:relative;
    display: inline-block
  }
  .voiceCon {
    width: 540px;
    display: inline-block;;
    vertical-align: top;
    padding-left: 10px;
    box-sizing: border-box;
    position: absolute;
    left: 100px;
    word-break: break-all;
  }
  .wifi-symbol {
      width: 38px;
      height: 38px;
      box-sizing: border-box;
      overflow: hidden;
      transform: rotate(135deg);
      position: absolute;
      left:15px;
      top:-7px
    }

    .wifi-circle {
      border: 2px solid #999999;
      border-radius: 50%;
      position: absolute;
    }

    .wifi-first {
      width: 5px;
      height: 5px;
      background: #cccccc;
      top: 35px;
      left: 35px;
    }

    .wifi-second {
      animation: fadeInOut 1s infinite 0.4s;
    }
    .pause-second {
      width: 20px;
      height: 20px;
      top: 30px;
      left: 30px;
    }
    .wifi-third {
      animation: fadeInOut 1s infinite 0.5s;
    }
    .pause-third {
      width: 25px;
      height: 25px;
      top: 25px;
      left: 25px;
    }
    @keyframes fadeInOut {
      0% {
        opacity: 0;
        /*初始状态 透明度为0*/
      }

      100% {
        opacity: 1;
        /*结尾状态 透明度为1*/
      }
    }
</style>
