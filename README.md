# vue-player
这是一个vue自定义播放器组件
## 组件使用示例:
### 1. 将项目组件克隆到components目录下:

### 2. 在项目中使用组件:

```
<template>
  <Player :resource='https://xxx.mp4' />
</template>
```
>> resource 为video的src加载路径
```
import Player from '@/components/vue-player'
components:{
  Player
}
```
