<template>
  <div class="contain-play" :class="{fullscreen: fullscreen}">
    <div ref="playertarget" class="player">
      <video id="video" ref="playertargetvideo" src="./lu.mp4" />
      <div ref="controls" class="controls">
        <div class="grey">
          <div ref="progress" class="progress" :style="{width:(currentTime/totalTime*100)+'%'}" />
          <div class="pointer" />
        </div>
        <div ref="gresstarget" class="grey-drag" @mousedown="progressmousedown">
          <div class="progress" :style="{width:constrolmove+'%'}" />
          <div class="pointer" />
        </div>
        <div class="pause">
          <div class="pause_left">
            <img v-if="playing" :src="resource" @click="switchPlay">
            <img v-else src="./img/play.png" @click="switchPlay">
          </div>
          <div class="pause_center">{{ currentTime | formateTime }} / {{ totalTime | formateTime }}</div>
          <div class="pause_right">
            <div class="fixed playbackRate-wrap">
              倍速
              <ul class="video-playbackRate">
                <li class="click-item" :class="{active: playbackRate === 0.5}" @click="switchplaybackRate(0.5)">0.5X</li>
                <li class="click-item" :class="{active: playbackRate === 1}" @click="switchplaybackRate(1)">1.0X</li>
                <li class="click-item" :class="{active: playbackRate === 1.5}" @click="switchplaybackRate(1.5)">1.5X</li>
                <li class="click-item" :class="{active: playbackRate === 2}" @click="switchplaybackRate(2)">2.0X</li>
                <li class="click-item" :class="{active: playbackRate === 2.5}" @click="switchplaybackRate(2.5)">2.5X</li>
              </ul>
            </div>
            <div class="fixed volume-wrap">
              <div ref="volumecard" class="volume-card">
                <div ref="volumetarget" class="volume" @mousedown="volumemousedown">
                  <div class="thumb" :style="{height: `${volumemove}%`}">
                    <div class="track" />
                  </div>
                </div>
              </div>
              <svg-icon v-if="videomuted" icon-class="sound-mute" class-name="fullscreen" @click="switchMuted" />
              <svg-icon v-else icon-class="sound" class-name="fullscreen" @click="switchMuted" />
            </div>
            <svg-icon v-if="fullscreen" icon-class="shrink" class-name="fullscreen fixed" @click="fullscreenSwitch" />
            <svg-icon v-else icon-class="expand" class-name="fullscreen fixed" @click="fullscreenSwitch" />
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  filters: {
    formateTime(data) {
      // 转换时长
      let h = Math.floor(data / 3600) || 0
      let m = Math.floor(data % 3600 / 60) || 0
      let s = Math.floor(data % 60) || 0
      // 转换格式
      h = h > 10 ? h : '0' + h
      m = m > 10 ? m : '0' + m
      s = s > 10 ? s : '0' + s
      return `${h}:${m}:${s}`
    }
  },
  props: {
    resource: {
      type: String,
      default: ''
    }
  },
  data() {
    return {
      playing: false,
      totalTime: 0,
      currentTime: 0,
      controls: false,
      constrolmove: 0,
      fullscreen: false,
      volumemove: '', // 音量键比例
      volumeTop: 0,
      volumeOffset: 0,
      videomuted: false, // 是否静音
      playbackRate: 0 // 播放倍速
    }
  },
  mounted() {
    this.initVideo()
    let timer = null
    this.$refs.playertarget.addEventListener('mouseenter', (e) => {
      this.$refs.controls.style.opacity = 1
    })
    this.$refs.playertarget.addEventListener('mousemove', (e) => {
      this.$refs.controls.style.opacity = 1
      this.$refs.playertargetvideo.style.cursor = ''
      if (timer) {
        clearTimeout(timer)
      }
      timer = setTimeout(() => {
        this.$refs.controls.style.opacity = 0
        this.$refs.playertargetvideo.style.cursor = 'none'
      }, 2000)
    })
    this.$refs.playertarget.addEventListener('mouseleave', (e) => {
      if (timer) {
        clearTimeout(timer)
      }
      setTimeout(() => {
        this.$refs.controls.style.opacity = 0
      }, 1000)
    })
  },
  methods: {
    // 初始化video
    initVideo() {
      this.$video = document.getElementById('video')
      // 监听缓存进度
      this.$video.addEventListener('progress', () => {
        console.log('progress')
      })
      // 监听播放进度
      this.$video.addEventListener('timeupdate', (e) => {
        this.currentTime = e.target.currentTime
      })
      // 监听播放结束
      this.$video.addEventListener('ended', (e) => {
        this.playing = false
      })
      // 监听视频能够播放
      this.$video.addEventListener('canplay', (e) => {
        this.volumemove = e.target.volume * 100
        this.totalTime = e.target.duration
        this.playbackRate = e.target.playbackRate
      })
      // 监听视频播放音量
      // this.$video.addEventListener('volumechange', (e) => {
      //   if (e.target.volume === 0) {
      //     this.videomuted = true
      //   }
      // })
      // 监听播放回放速率
      this.$video.addEventListener('ratechange', (e) => {
        this.playbackRate = e.target.playbackRate
      })
    },
    // 视频暂停播放功能
    switchPlay() {
      this.playing = !this.playing
      if (this.$video.paused) {
        try {
          this.$video.play()
        } catch {
          this.playing = !this.playing
        }
      } else {
        try {
          this.$video.pause()
        } catch {
          this.playing = !this.playing
        }
      }
    },
    // 隐藏拖动进度条
    hiddenprogress() {
      if (this.$refs.gresstarget) {
        this.$refs.gresstarget.style.opacity = 0
        document.removeEventListener('mousemove', this.dragprogress)
        this.$video.currentTime = this.constrolmove * this.totalTime / 100
        if (this.$video.paused) {
          try {
            this.$video.play()
            this.playing = true
          } catch (e) {
            console.log(e)
          }
        }
      }
    },
    // 根据鼠标位置拖动进度条
    dragprogress(e) {
      const clientX = e.clientX
      const offLeft = this.$refs.playertarget.offsetLeft
      const clientWidth = this.$refs.playertarget.clientWidth
      const stywidth = clientX - offLeft
      // this.$refs.progress.style.transition = 'unset'
      // this.$refs.progress.style.width = `${stywidth / clientWidth * 100}%` // 进度条过度动画
      this.constrolmove = stywidth / clientWidth * 100
      // setTimeout(() => {
      //   this.$refs.progress.style.transition = '' // 进度条过度动画
      // })
    },
    // 鼠标按下进度条
    progressmousedown(e) {
      this.constrolmove = this.currentTime / this.totalTime * 100
      this.$refs.gresstarget.style.opacity = 1
      document.addEventListener('mouseup', this.hiddenprogress, { once: true })
      document.addEventListener('mousemove', this.dragprogress)
    },

    // 根据鼠标位置拖动音量条
    dragvolume(e) {
      const clientY = e.clientY
      if ((clientY >= this.volumeTop - this.volumeOffset) && (clientY <= this.volumeTop)) {
        this.volumemove = (this.volumeTop - clientY) / this.volumeOffset * 100
        this.$video.volume = (this.volumeTop - clientY) / this.volumeOffset
        if (this.volumeTop - clientY === 0) {
          this.videomuted = true
        } else {
          this.videomuted = false
        }
      }
    },
    // 获取当前元素在屏幕上的位置
    getDomeTop(obj) {
      let DomTop = 0
      while (obj) {
        DomTop += obj.offsetTop
        obj = obj.offsetParent
      }
      return DomTop
    },
    // 取消音量拖动监听
    clearvolumedrag() {
      document.removeEventListener('mousemove', this.dragvolume)
      this.$refs.volumecard.style.display = ''
    },
    // 鼠标按下音量键
    volumemousedown() {
      this.$refs.volumecard.style.display = 'block'
      this.volumeTop = this.getDomeTop(this.$refs.volumetarget)
      this.volumeOffset = this.$refs.volumetarget.offsetHeight
      document.addEventListener('mouseup', this.clearvolumedrag, { once: true })
      document.addEventListener('mousemove', this.dragvolume)
    },
    // 切换静音
    switchMuted() {
      var sta = this.$video.muted
      if (sta === true) {
        this.$video.muted = false
        this.videomuted = false
      } else {
        this.$video.muted = true
        this.videomuted = true
      }
    },
    // 切换倍速
    switchplaybackRate(rate) {
      this.$video.playbackRate = rate
    },

    // 全屏播放器
    fullscreenSwitch() {
      const element = document.documentElement
      if (this.fullscreen) {	// fullscreen全屏标示符
        if (document.exitFullscreen) {
          document.exitFullscreen()
        } else if (document.webkitCancelFullScreen) {
          document.webkitCancelFullScreen()
        } else if (document.mozCancelFullScreen) {
          document.mozCancelFullScreen()
        } else if (document.msExitFullscreen) {
          document.msExitFullscreen()
        }
      } else {
        if (element.requestFullscreen) {
          element.requestFullscreen()
        } else if (element.webkitRequestFullScreen) {
          element.webkitRequestFullScreen()
        } else if (element.mozRequestFullScreen) {
          element.mozRequestFullScreen()
        } else if (element.msRequestFullscreen) {
          // IE11
          element.msRequestFullscreen()
        }
      }
      this.fullscreen = !this.fullscreen
    }
  }
}
</script>

<style lang="scss" scoped>
@media screen and (max-width:1366px){
  .contain-play .player{
    .controls{
      height: 68px;
      .pause{
        font-size: 12px;
        .pause_left{
          padding-left: 20px;
          img{
            width: 46px;
            height: 46px;
            margin: 0 28px;
          }
        }
      }
    }
  }
}
@media screen and (min-width:1367px){
  .contain-play .player{
    .controls{
      height: 95px;
      .pause{
        font-size: 17px;
        .pause_left{
          padding-left: 28px;
          img{
            width: 64px;
            height: 64px;
            margin: 0 40px;
          }
        }
      }
    }
  }
}
.contain-play {
  height: 100%;
  width: 100%;
  display: flex;
  justify-content: center;
  align-items:center;
  overflow: hidden;
  &.fullscreen{
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: #000;
  }
  .player {
    width: 100%;
    height: 100%;
    user-select: none;
    position: relative;
    display: inline-block;
    font-size: 0;
    #video{
      width: 100%;
      height: 100%;
    }
    .controls {
      transition: opacity .5s;
      opacity: 0;
      position: absolute;
      bottom: 0;
      background: linear-gradient(0deg, #313131 0%, rgba(49, 49, 49, 0.31) 100%);
      width: 100%;
      .grey{
        position: absolute;
        top:0;
        transform: translateY(-50%);
        width: 100%;
        background: #000;
        display: flex;
        height: 4px;
        .progress{
          // transition: width .35s linear; // 进度条过度动画
          background: #5AC75E;
        }
        .pointer{
          height: 8px;
          width: 8px;
          border-radius: 100%;
          background: #fa940c;
          transform: translate(-2px, -2px);
        }
      }
      .grey-drag{
        position: absolute;
        transform: translateY(-50%);
        top: 0;
        width: 100%;
        background: #000;
        display: flex;
        height: 8px;
        opacity: 0;
        .progress{
          background: #5AC75E;
        }
        .pointer{
          height: 12px;
          width: 12px;
          border-radius: 100%;
          background: #fa940c;
          transform: translate(-2px, -2px);
        }
      }
      .pause{
        display: flex;
        height: 100%;
        color: white;
        img{
          cursor: pointer;
        }
        .pause_left,.pause_center,.pause_right{
          display: flex;
          align-items: center;
          .fixed{
            padding: 20px;
            position: relative;
            &.volume-wrap{
              position: relative;
              &:hover{
                .volume-card{
                  display: block;
                }
              }
              .volume-card{
                display: none;
                padding:0 20px;
                position: absolute;
                top: 0px;
                left: 50%;
                height: 120px;
                transform: translate(-50%, -100%);
                .volume{
                  width: 7px;
                  height: 100%;
                  background: #ccc;
                  border-radius: 5px;
                  display: flex;
                  flex-direction: row-reverse;
                  align-content: flex-end;
                  flex-wrap: wrap;
                  cursor: pointer;
                  .thumb{
                    width: 100%;
                    background: #5ac75e;
                    border-radius: 5px;
                    position: relative;
                    .track{
                      position: absolute;
                      top:0;
                      width: 11px;
                      height: 11px;
                      transform: translate(-2px, -50%);
                      border-radius: 100%;
                      background: #fa940c;
                    }
                  }
                }
              }
            }
            &.playbackRate-wrap{
              cursor: pointer;
              &:hover{
                .video-playbackRate{
                  display: block;
                }
              }
              .video-playbackRate{
                display: none;
                margin: 0;
                padding: 0;
                list-style: none;
                position: absolute;
                width: 150px;
                padding: 14px 0;
                color: #ddd;
                border-radius: 5px;
                background-color: rgba(26,26,26,.6);
                top: 0;
                left: -30px;
                transform: translateY(-100%);
                .click-item{
                  text-align: center;
                  height: 47px;
                  letter-spacing: 3px;
                  line-height: 47px;
                  cursor: pointer;
                  transition: transform .5s;
                  &:hover{
                    transform: scale(1.2);
                  }
                  &.active{
                    color: #5ac75e;
                  }
                }
              }
            }
          }
        }
        .pause_center{
          flex:1;
        }
        .pause_right{
          padding-right: 30px;
          .fullscreen{
            margin: 20px;
            padding: 0;
            font-size: 32px;
            cursor: pointer;
          }
        }
      }
    }
  }
}
</style>
