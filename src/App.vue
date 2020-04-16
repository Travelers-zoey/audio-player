<template>
  <div id="app">
    <!-- 背景图片 -->
      <div
        class="bg"
        style="background-image: url(https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1586685609637&di=dcea9b2d3d06e7b6ad7e8a655a3bcfda&imgtype=0&src=http%3A%2F%2Fpic1.win4000.com%2Fmobile%2F8%2F5848ef53428b4.jpg)"
      ></div>
      <div
        class="bg-blur bg"
        style="background-image: url(https://timgsa.baidu.com/timg?image&quality=80&size=b9999_10000&sec=1586685609637&di=dcea9b2d3d06e7b6ad7e8a655a3bcfda&imgtype=0&src=http%3A%2F%2Fpic1.win4000.com%2Fmobile%2F8%2F5848ef53428b4.jpg)"
      ></div>

      <!-- 主体部分 -->
      <div class="music-wrapper">
        <!-- 标题信息 -->
        <div class="topBar">
          <div class="back"><i class="el-icon-arrow-left"></i></div>
          <!-- <i class="el-icon-arrow-left"></i> -->
          <div class="info">
            <div class="title">
              {{name}}
            </div>
            <div class="author">
              {{author}}
            </div>
          </div>
          <div class="share"><i class="el-icon-close"></i></div>
          <!-- <i class="el-icon-close"></i> -->
        </div>

        <!-- 中间的转盘 -->
        <div class="main" :class="{playing: isPlaying}">
          <!-- <div class="CD" v-bind:style="'transform: rotate(' + rotateDeg + 'deg)'"></div> -->

          <!-- 播放杆 -->
          <img src="./assets/img/player_bar.png" class="play_bar" alt="">
          <div class="CD"  v-bind:style="'transform: rotate(' + rotateDeg + 'deg)'">
            <!-- 黑胶片 -->
            <img src="./assets/img/disc.png" class="CD disc" /> 
            <img :src="bgImg" class="CD cover" alt=""/>
          </div>
        </div>

        <!-- 进度条 -->
        <div class="progress">
          <div class="curTime">
            {{transferSecToTime(audioInfo.currentTime)}}
          </div>
          <div class="progress-line">
            <div class="whole-line">
              <div
                class="already-line"
                :style="'width:' + progress + '%'"
              ></div>
            </div>
            <div class="play-point" :style="'left:' + progress + '%'"></div>
          </div>
          <div class="AllTime">{{transferSecToTime(audioInfo.duration)}}</div>
        </div>

      <!-- 音频播放器 -->
      <audio id="player" v-on:canplay="handleCanPlay" v-on:timeupdate="handleTimeUpdate"></audio>

        <!-- 控制条 -->
        <div class="toolBar">
          <div class="heart"><i class="iconfont">&#xe68d;</i></div>
          <div class="last" v-on:click="playPreSong">
            <i class="iconfont">&#xe6e1;</i>
          </div>
          <div v-if="!isPlaying" class="play" v-on:click="playMusic">
            <i class="iconfont">&#xe717;</i>
          </div>
          <div v-if="isPlaying" class="pause" v-on:click="pauseMusic">
            <i class="iconfont">&#xe643;</i>
          </div>
          <div class="next" v-on:click="playNextSong">
            <i class="iconfont">&#xe718;</i>
          </div>
          <div class="comments" v-on:click="isVisible=true">
            <i class="iconfont">&#xe6d3;</i>
            <span>{{comments.length}}</span>
          </div>
        </div>
      </div>

      <!-- 评论列表 -->
      <div class="comment-list" id="comments-box" v-if="isVisible">
        <div class="comment-list__back" v-on:click="isVisible=false">
          <!-- <i class="iconfont">&#xe61e;</i> -->
          <i class="el-icon-arrow-left"></i>
        </div>
        <div class="comment-list__count">
          评论（{{comments.length}}）
        </div>
        <div class="comment-list__item" v-for="(comment,index) in comments" :key="index">
          <div class="comment-list__item-meta">
            <div class="comment-list__item-meta__name">
              {{comment.name}}
            </div>
            <div class="comment-list__item-meta__date">
              {{comment.createdAt}}
            </div>
          </div>
          <div class="comment-list__item-comment">
            {{comment.comment}}
          </div>
        </div>
        <div class="comment-pub" >
          <zoey-input v-model="inputtedComment" autofocus="true" @input="msg => this.inputtedComment = msg" placeholder="发表评论"></zoey-input>
          <zoey-button v-if="isEmpty" type="primary"  @click="addComment">发 送</zoey-button>
          <zoey-button v-else type="primary" disabled @click="addComment">发 送</zoey-button>
        </div>
      </div>
  </div>
</template>

<script>
// import HelloWorld from './components/HelloWorld.vue'
import ZoeyButton from './components/ZoeyButton'
import ZoeyInput from './components/ZoeyInput'

export default {
  name: 'App',
  components: {
    ZoeyButton,
    ZoeyInput
  },
  created() {
    // 获取歌单
    fetch('https://jirengu.github.io/data-mock/huawei-music/music-list.json')
      .then(res => res.json())
      .then(data => {
        this.songList = data;
        // 从歌单里面获取歌曲名字和作者
        var player = document.getElementById('player');
        player.src = data[this.currentIndex].url;
        this.name = data[this.currentIndex].title;
        this.author = data[this.currentIndex].author;
      })
  },
  // 数据定义
  data() {
    return {
      name: '',
      author: '',
      bgImg: 'https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=3443888898,3749783847&fm=26&gp=0.jpg',
      // 音乐歌单
      songList: [],
      isPlaying: false,
      // 封面旋转角度
      rotateDeg: 0,
      // 定时器
      clockNum: 0,
      // 弹窗是否显示
      isVisible: false,
      audioInfo: {
        duration: 0,
        currentTime: 0
      },
      // 评论
      comments: [
        {
          name: 'Travelers',
          comment: '我也羡慕那些一沾枕头就能安睡和那些决心放手之后就不再回头的人',
          createdAt: '2020/3/12'
        },
        {
          name: '山河回望',
          comment: '与其热闹着引人夺目，步步紧逼，不如趋向做一个人群之中真实自然的人，不张扬，不虚饰，随时保持退后的位置，心有所定，只是专注做事',
          createdAt: '2020/3/29'
        },
        {
          name: 'zoey',
          comment: '与其热闹着引人夺目，步步紧逼，不如趋向做一个人群之中真实自然的人，不张扬，不虚饰，随时保持退后的位置，心有所定，只是专注做事',
          createdAt: '2020/4/12'
        }
      ],
      // 输入框的评论
      inputtedComment: '',
      // 播放的音乐下标，根据下标记录播放哪一首
      currentIndex: 0,
      isEmpty: false
    }
  },
  computed: {
    progress() {
      return (
        (this.audioInfo.currentTime / (this.audioInfo.duration || 1)) *
        100
      ).toFixed(1);
    }
  },
  methods: {
    playOrPause() {
      this.$refs.audio.play();
    },
    startRotate() {
      // 定义旋转间隔，单位ms，这里设定为每10ms旋转一次，间隔时间取值太大的话会使得旋转看起来有卡顿感
      var rotateDuration = 10;
      // 我们设定每60s封面旋转一圈，也就是360°
      // 这里计算的是每次旋转间隔需要旋转的度数
      var rotateStep = (360 / 60) * (rotateDuration / 1000);
      // 保存this，很关键，定时器内部会用到
      var that = this;

      // 设定定时器，自动旋转
      var clockNum = setInterval(function () {
        // 旋转角度增加
        that.rotateDeg += rotateStep;
      }, rotateDuration);

      // 保存定时器num，暂停的时候需要
      this.clockNum = clockNum;
    },
    playMusic() {
      var player = document.getElementById('player');
      // player.src = this.songList[this.currentIndex].url;
      player.play();
      this.isPlaying = true;
      // 封面开始旋转
      this.startRotate();
    },
    // 暂停按钮处理函数
    pauseMusic() {
      var player = document.getElementById('player');
      player.pause();
      this.isPlaying = false;
      // 封面暂停旋转
      this.pauseRotate();
    },
    pauseRotate() {
      // 清除定时器，封面也就不会旋转了
      clearInterval(this.clockNum);
    },
    playPreSong() {
      this.pauseRotate();
      this.currentIndex = (this.songList.length + this.currentIndex -1) % this.songList.length;
      var player = document.getElementById('player');
      player.src = this.songList[this.currentIndex].url;
      this.name = this.songList[this.currentIndex].title;
      this.author = this.songList[this.currentIndex].author;
      player.play();
      this.isPlaying = true;
      // 封面开始旋转
      this.startRotate();
    },
    playNextSong() {
      this.pauseRotate();
      this.currentIndex = (this.songList.length + this.currentIndex +1) % this.songList.length;
      var player = document.getElementById('player');
      player.src = this.songList[this.currentIndex].url;
      this.name = this.songList[this.currentIndex].title;
      this.author = this.songList[this.currentIndex].author;
      player.play();
      this.isPlaying = true;
      // 封面开始旋转
      this.startRotate();
    },
    // 监听音频可以播放时的事件
    handleCanPlay(event) {
      this.audioInfo.duration = event.target.duration;
    },
    // 监听音频播放位置发送改变的事件
    handleTimeUpdate(event) {
      this.audioInfo.currentTime = event.target.currentTime;
    },
    transferSecToTime(time) {
      var tempTime = Math.floor(time);
      var min = 0;
      var sec = 0;
      if (tempTime >= 60) {
        min = Math.floor(tempTime / 60);
        sec = tempTime % 60;
      } else {
        sec = tempTime;
      }
      if (min < 10) min = '0' + min;
      if (sec < 10) sec = '0' + sec;
      return min + ':' + sec;
    },
    addComment() {
      var createdAt = new Date().toLocaleDateString();
      var name = '我是全菜';
      this.comments.push({
        name: name,
        comment: this.inputtedComment,
        createdAt: createdAt
      });
      this.inputtedComment = '';
      this.isEmpty = false;

      // this.$nextTick(function(){
      // var div = document.getElementById('comment-box');
      //   div.scrollTop = div.scrollHeight;
      // })
    }
  },
}
</script>

<style>
@font-face {
  font-family: 'iconfont';  /* project id 1622128 */
  src: url('http://at.alicdn.com/t/font_1622128_z6hlqa9uqe.eot');
  src: url('http://at.alicdn.com/t/font_1622128_z6hlqa9uqe.eot?#iefix') format('embedded-opentype'),
  url('http://at.alicdn.com/t/font_1622128_z6hlqa9uqe.woff2') format('woff2'),
  url('http://at.alicdn.com/t/font_1622128_z6hlqa9uqe.woff') format('woff'),
  url('http://at.alicdn.com/t/font_1622128_z6hlqa9uqe.ttf') format('truetype'),
  url('http://at.alicdn.com/t/font_1622128_z6hlqa9uqe.svg#iconfont') format('svg');
}
.iconfont{
    font-family:"iconfont" !important;
    font-size:50px;font-style:normal;
    -webkit-font-smoothing: antialiased;
    -webkit-text-stroke-width: 0.2px;
    -moz-osx-font-smoothing: grayscale;
}
body {
  transform: scale(0.5);
  transform-origin: left top;
  width: 720px;
  height: 1280px;
  padding: 0;
  margin: 0;
}

#app {
  width: 100%;
  height: 100%;
  position: relative;
  overflow: hidden;
}
.bg {
    /* background-image: url(https://s2.ax1x.com/2020/01/27/1uPIAS.jpg); */
    width: 100%;
    height: 100%;
    position: absolute;
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;
    filter: blur(15px);
    transform: scale(1.1);
}
.bg-blur {
    /* background-image: url(https://s2.ax1x.com/2020/01/27/1uPIAS.jpg); */
    width: 100%;
    height: 100%;
    position: absolute;
    background-repeat: no-repeat;
    background-position: center;
    background-size: cover;
    filter: blur(100px);
    transform: scale(1.1);
}

.back {
  font-size: 55px;
}

.music-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
}
.topBar {
  margin: 0 4%;
  height: 10%;
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-size: 35px;
  color: #000;
}

.topBar .info {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
}

.topBar .info .author {
  color: #000;
  font-size: 28px;
}

.topBar .share i {
  font-size: 60px;
}
.main {
  margin: 0 4%;
  height: 60%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 32px;
}

.main .CD {
  height: 400px;
  width: 400px;
  background-color: white;
  border-radius: 50%;
  background: center;
  /* background-image: url(https://ss1.bdstatic.com/70cFuXSh_Q1YnxGkpoWK1HF6hhy/it/u=3443888898,3749783847&fm=26&gp=0.jpg); */
}
/* 黑胶片 */
.disc {
  position: absolute;
  z-index: 3;
}
.cover {
  position: absolute;
  z-index: 2;
}

/* 播放杆 */
.play_bar {
  position: absolute;
  left: 50%;
  top: 230px;
  z-index: 4;
  transform: rotate(-25deg);
  transform-origin: 12px 12px;
  transition: 1s;
}
/* 播放杆 转回去 */
.main.playing .play_bar {
  transform: rotate(0);
}
.progress {
  /* color: #8e857b; */
  font-size: 20px;
  margin: 0 4%;
  height: 15%;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.progress .progress-line {
  width: 500px;
  position: relative;
  display: flex;
  align-items: center;
  color: #8e857b;
}

.progress .progress-line .already-line {
  position: absolute;
  height: 4px;
  border-radius: 2px;
  background-color: #fff;
}

.progress .progress-line .play-point {
  width: 18px;
  height: 18px;

  transition: all .2s linear;
  border-radius: 50%;
  position: absolute;
  background-color: #fefffe;
}

.progress .progress-line .whole-line {
  width: 100%;
  height: 4px;
  border-radius: 2px;
  background-color: #8c8379;
}
.curTime, .allTime {
  width : 55px;
}
.toolBar {
  color: #fefffe;
  margin: 0 4%;
  height: 15%;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.toolBar i {
  cursor: pointer;
}

.toolBar .play .iconfont {
  font-size: 70px;
}

.toolBar .pause .iconfont {
  font-size: 70px;
}

.comments {
  position: relative;
}

.comments span  {
  font-size: 16px;
  position: absolute;
  right: 0;
}

.comment-list {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  overflow: auto;
  background: rgba(0, 0, 0, 0.5);
}

.comment-list__back {
  position: absolute;
  height: 50px;
  line-height: 50px;
  cursor: pointer;
}

.comment-list__back i {
  font-size: 24px;
  color: white;
  margin-left: 20px;
}

/* 评论 */
.comment-list__count {
  color: rgb(255, 255, 255);
  font-size: 32px;
  font-weight: 600;
  text-align: center;
  height: 50px;
  line-height: 50px;
}

.comment-list__item {
  padding: 0 20px 20px;
  color: rgb(209, 202, 202);
  font-size: 24px;
  margin-bottom: 20px;
  border-bottom: 1px solid white;
}

.comment-list__item:first-child {
  padding-top: 20px;
}

.comment-list__item-meta {
  margin-bottom: 10px;
}

/* 日期字样式 */
.comment-list__item-meta__date {
  font-size: 22px;
  color: #a09797;
}

.comment-list__item-meta__name {
  font-size: 28px;
  font-weight: 500;
  color: rgb(231, 225, 225);
}
.comment-pub {
  width: 92%;
  position: fixed;
  display: flex;
  background-color: #1c1b1e;
  padding: 0 4%;
  height: 60px;
  bottom: 0;
  justify-content: space-between;
  align-items: center;
}

.comment-pub input {
  width: 70%;
  height: 100%;
  font-size: 20px;
  border: none;
  outline: none;
  background-color: #1c1b1e;
  color: #fff;
}

.comment-pub .pub-button {
  cursor: pointer;
  display: flex;
  color: #464648;
  align-items: center;
  height: 100%;
  font-size: 20px;
  color: #a9a39c;
}
</style>
