# vue-player
这是一个vue自定义播放器组件
## 组件使用示例:
### 1. 将项目组件克隆到components目录下:
>``` 需要安装hls.js 支持m3u8
### 2. 在项目中使用组件:

```
<template>
  <Player ref="m3u8VideoPlay" />
</template>
```
>> resource 为video的src加载路径
```
import Player from '@/components/vue-player'
components:{
  Player
},
methods: {
  videoPlay() {
    this.$refs.m3u8VideoPlay.videoPlay({
      videoUlr: '',
      ...
    })
  }
}
```
