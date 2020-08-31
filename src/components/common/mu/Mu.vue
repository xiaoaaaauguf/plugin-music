<template>
  <div>
    <!--    歌曲菜单-->
    <div class="list_box" v-if="listIsDis">
      <!--              本地歌曲弹窗-->
      <transition name="music_alert">
        <span class="music_alert" v-if="musicAlertState">{{musicAlertVal}}</span>
      </transition>
      <!--      退出键-->
      <div class="list_close" @click="DisList">x</div>
      <div class="music_list">
        <!--          左边-->
        <div class="list_l">
          <!-- 菜单列表-->
          <ul class="music_type">
            <li v-for="item in musicTypeList" @click="_getMusicType(item.id,true)"
                :class="{type_active:item.id==thisMusicType}">{{item.name}}
            </li>
          </ul>
          <!--           搜索栏 -->
          <div class="list_title">
            <span style="font-size: 14px;">歌曲列表</span>
            <img :src="musicStateButton" alt="" class="music_state" @click="MusicStateChange">
            <div class="music_search_box">
              <input type="text" class="music_search" v-model="musicSearchVal" placeholder="搜索歌曲" @blur="searchResult">
              <transition name="music_search">
                <ul class="search_list" v-if="musicSearchVal!=''">
                  <li v-for="item in musicSearchList" @click="ListAdd(item)">
                    <span class="music_search_name">{{item.name}}</span>
                    <span class="music_search_ar">{{item.artists[0].name}}</span>
                  </li>
                </ul>
              </transition>
            </div>
          </div>
          <!-- 歌手歌曲专辑 -->
          <div class="music_ul_title">
            <span>歌曲</span><span>歌手</span><span>专辑</span>
          </div>
          <!-- 歌曲列表 -->
          <ul class="_list">
            <li v-for="(item,index) in thisMusicList" @mouseover="ButtonActive(index)"
                @dblclick="ListPlay((thisListPage-1)*10+index)">
              <div class="this_music_shlter" v-if="(thisListPage-1)*10+index==thisMusicIndex"></div>
              <span>{{item.name}}</span><span>{{item.ar[0].name}}</span><span>{{item.al.name}}</span>
              <!--                播放添加按钮-->
              <div class="music_button" v-if="listButtonActiveIndex==index">
                <div class="list_play" title="播放这首歌" :style="{backgroundImage:'url('+listPlay+')'}"
                     @click="ListPlay((thisListPage-1)*10+index)"></div>
                <div class="list_play" title="添加到 My Songs" :style="{backgroundImage:'url('+add+')'}"
                     @click="ListAdd(item)" v-if="thisMusicType!=-1"></div>
              </div>
            </li>
          </ul>
          <!-- 翻页按钮 -->
          <div class="list_page">
            <div class="page_last" v-if="thisListPage!=1" @click="ListChange(true)"> <</div>
            <div class="page_next" v-if="thisListPage!=Math.ceil(musicList.length/10)"
                 @click="ListChange(false)"> >
            </div>
          </div>
        </div>
        <!--          右边-->
        <!--        <div class="list_r">-->
        <!--        </div>-->
      </div>
    </div>
    <!-- 播放器小方块 -->
    <div class="boxx" :class="{boxx_active:disActive}">
      <!-- 圈圈 -->
      <div class="pan" :style="{backgroundImage:'url('+pan+')'}" :class="{pan_active:disActive}" @click="DisActive">
        <img :src="musicImg" class="pan_c" alt=""/>
      </div>
      <!-- 方块放大内容 -->
      <div class="box" :style="{backgroundImage:'url('+musicImg+')'}" :class="{box_active:disActive}"
           @dblclick="DisList">
        <div class="music_1" :style="{backgroundImage:'url('+musicImg+')'}"
             :class="{shlter_active:disActive}"></div>
        <div class="music_2" :style="{backgroundImage:'url('+musicImg+')'}"
             :class="{shlter_active:disActive}"></div>
        <div class="music_3"></div>
        <div class="music_dis">
          <div class="dis_list" @click="DisList">···</div>
          <!-- 标题 -->
          <p class="music_title">{{musicTitle}}</p>
          <!-- 歌手 -->
          <p class="music_intro">歌手: {{musicName}}</p>
          <!-- 歌词 -->
          <ul class="music_words">
            <div class="music_words_box" :style="{top:wordsTop+'px'}">
              <li v-for="(item,index) in musicWords" class="music_word" :class="{word_highlight:wordIndex==index}">
                {{item}}
              </li>
            </div>
          </ul>
        </div>
        <!--        进度条暂停按钮-->
        <div class="control_box">
          <!--          暂停按钮-->
          <div class="control_button">
            <img :src="playIcon" alt="" class="control_icon">
          </div>
          <!--          进度条-->
          <div class="progress_b">
            <div class="progress_c" :style="{width:currentProgress}">
              <div class="progress_circle">
                <div class="progress_circle_c"></div>
              </div>
            </div>
          </div>
        </div>
      </div>
      <!--            播放声音-->
      <audio id="music" autoplay="autoplay" :src="musicUrl" @play="musicPlay" @pause="musicPause"
             @ended="musicEnded"></audio>
    </div>
  </div>
</template>

<script>
  import {getWords, getMusicInfo, getMusicUrl, getHotMusic, getSearchSuggest, getHotTalk} from './api/music'
  import pan from './img/pan.png'
  import play from './img/play.png'
  import pause from './img/pause.png'
  import add from './img/add.png'
  import shlter from './img/list_pan.png'
  import listPlay from './img/list_play_hover.png'
  import state0 from './img/state_0.png'
  import state1 from './img/state_1.png'
  import talkicon1 from './img/talkicon1.png'
  import talkicon2 from './img/talkicon2.png'

  export default {
    name: 'Mu',
    data() {
      return {
        pan,  //圈圈边框图片
        listPlay, //列表播放图片
        add,  //列表添加图片
        o: 0,
        top: 0,
        disActive: false,  //点击圈圈
        listIsDis: true,  //点击三点
        musicTitle: '', //歌名
        musicName: '',  //歌手
        wordsTop: 0,  //歌词的top
        wordIndex: 0,   //歌词位置
        musicWords: [], //歌词
        wordTime: [],  //歌词显示时间
        musicUrl: '', //音乐地址
        musicList: [], //歌曲信息
        thisMusicIndex: 1, //选择第几首歌
        myMusicList: [],   //存储在本地   可以开始判断有没有 让用户一开始就听这个列表
        musicImg: '',  //歌曲图片
        hotTalkList: [],  //歌曲评论信息
        currentProgress: '0%',  //播放百分比
        playIcon: play,  //暂停按钮
        //菜单部分
        musicTypeList: [
          {name: '热歌榜', id: 1},
          {name: '新歌榜', id: 0},
          {name: '飙升榜', id: 3},
          {name: '嘻哈榜', id: 18},
          {name: 'My Songs', id: -1}
        ],  //列表项
        thisMusicType: -1,  //列表点击项
        notPlay: [],
        musicStateButton: state1, //单曲循环按钮
        musicState: 0,   //0列表循环,1单曲循环
        musicSearchVal: '',   //搜索内容
        musicSearchList: [],  //搜索结果列表
        musicAlertVal: '',  //弹窗内容
        musicAlertState: false,  //是否弹窗
        myMusicList: [], //存储在本地 可以开始判断有没有 让用户一开始就听这个列表
        musicAlertTimer: '',
        playState: true,  //播放状态
        thisListPage: 1, //当前列表页数
        listButtonActiveIndex: -1,  //鼠标经过歌曲列表效果
        //新增歌词评论
        hotTalkList: [],
      }
    },
    mounted() {
      this._getMusicType(1)
      this.Player()
    },
    watch: {
      musicSearchVal() {
        if (this.musicSearchVal == '') {
          this.musicSearchList = []
        } else {
          getSearchSuggest(this.musicSearchVal).then(res => {
            if (res.data.result.songs == undefined) {
              this.musicSearchList = []
            } else {
              this.musicSearchList = res.data.result.songs
            }
          })
        }
      }
    },
    computed: {
      thisMusicList() {
        return [...this.musicList].splice((this.thisListPage - 1) * 10, 10);  //分页
      }
    },
    methods: {
      musicPlay() {
        this.playIcon = pause
      },
      musicPause() {
        this.playIcon = play
      },
      musicEnded() {
      },
      DisActive() {
        this.disActive = !this.disActive
      },
      DisList() {
        this.listIsDis = !this.listIsDis
      },
      ButtonActive(id) {
        this.listButtonActiveIndex = id;
      },
      ListChange(isLast) {
        if (isLast) {
          this.thisListPage--;
        } else {
          this.thisListPage++;
        }
      },
      MusicAlert(val) {
        this.musicAlertState = true;
        this.musicAlertVal = val;
        clearTimeout(this.musicAlertTimer);
        this.musicAlertTimer = setTimeout(() => {
          this.musicAlertState = false;
          this.musicAlertVal = '';
        }, 2000);
      },
      searchResult() {
        if (this.musicSearchList.length == 0) {
          this.MusicAlert('未找到该歌曲')
        }
      },
      MusicStateChange() {
        if (this.musicState == 0) {
          this.musicState = 1;
          this.musicStateButton = state0;
          this.MusicAlert('已切换为单曲循环模式');
        } else {
          this.musicState = 0;
          this.musicStateButton = state1;
          this.MusicAlert('已切换为列表循环模式');
        }
      },
      ListAdd(obj) {
        getMusicInfo(obj.id).then((res) => {
          this.musicSearchVal = '';
          if (this.myMusicList.length == 0) {
            this.myMusicList = [res.data.songs[0]];
            console.log(res.data.songs[0])
            this._getMusicType(-1);
            //第一次搜索直接播放
          } else {
            this.myMusicList.push(res.data.songs[0]);
            //提示已经添加进去
          }
          this.MusicAlert('添加成功');
        })
      },
      _getMusicType(id, k) {
        //歌曲列表点击项高亮
        if (this.thisMusicType != id) {

          this.notPlay = []
          if (id == -1) {
            if (this.myMusicList.length != 0) {
              this.musicList = this.myMusicList
              this.thisMusicType = id
              this.thisMusicIndex = 0
              this.thisListPage = 1
              this._getInfo()
              this.update()
            } else {
              this.MusicAlert('没有歌曲,需要自己添加')
            }
          } else {
            //获取热门歌曲
            getHotMusic(id).then(res => {
              this.musicList = res.data.playlist.tracks.splice(0, 200)//我一直没搞懂
              this.thisMusicIndex = 0
              this.thisMusicType = id
              this.thisListPage = 1
              this._getInfo()
              if (k) {
                $('#music')[0].play()
                this.update()
              }
            })
          }
        }
      },
      _getInfo() {
        //获取歌曲url
        getMusicUrl(this.musicList[this.thisMusicIndex].id).then(res => {
          if (!res.data.data[0]) {
            if (this.notPlay.length != this.musicList.length) {
              let nextIndex = this.thisMusicIndex + 1
              if (this.notPlay.indexOf(this.thisMusicIndex) == -1) {
                this.notPlay.push(this.thisMusicIndex)
              }
              this.MusicAlert(`${this.musicList[this.thisMusicIndex].name}因为一些原因不能播放`)
              this.ListPlay(nextIndex)  //寻找下一首直到找到
            } else {
              console.log('not')
              this.MusicAlert('此列表所有歌都不能播放')
            }
          } else {
            this.musicUrl = res.data.data[0].url.replace('http://', 'https://')
            this.musicImg = this.musicList[this.thisMusicIndex].al.picUrl.replace('http://', 'https://') + '?param=300y300'
            this.musicTitle = this.musicList[this.thisMusicIndex].name

            //搞不懂为啥要搞这些花的
            // let name_arr=[]
            // this.musicList[this.thisMusicIndex].ar.forEach(i=>{
            //   name_arr.push(i.name)
            // })
            // this.musicName=name_arr.join('/')
            this.musicName = this.musicList[this.thisMusicIndex].ar[0].name
//获取歌词
            getWords(this.musicList[this.thisMusicIndex].id).then(res => {
              if (!res.data.nolyric) {
                let info = this.Cut(res.data.lrc.lyric)
                this.musicWords = info.wordArr
                this.wordTime = info.timeArr
              }
            })
            //获取歌曲热门评论
            getHotTalk(this.musicList[this.thisMusicIndex].id).then(res => {
              // let count=0
              // this.hotTalkList=res.data.hotComments.splice(0,3)
              // this.hotTalkList.forEach(e => {
              //   count += e.content.length;
              //   e.user.avatarUrl = e.user.avatarUrl + '?param=50y50';
              // })
              // if (count >= 200) {
              //   this.hotTalkList = this.hotTalkList.slice(0, 2);
              // }
            })
          }
        })
      },
      // 歌词截取函数
      Cut(str) {
        let timeArr = []
        let wordArr = []
        timeArr = str.split('[')
        timeArr.splice(0, 1)
        for (let i = 0; i < timeArr.length; i++) {
          let cut = timeArr[i].split(']')
          let time = cut[0].split('.')[0].split(':')
          timeArr[i] = Number.parseInt(time[0]) * 60 + Number.parseInt(time[1])
          timeArr[i] = isNaN(timeArr[i]) ? 0 : timeArr[i]; //处理NaN
          wordArr[i] = cut[1]
          // wordArr[i] = this.Rtrim(this.Ltrim(cut[1]));
        }
        return {timeArr, wordArr}
      },
      ListPlay(id) {
        $('#music')[0].play()
        if (this.thisMusicIndex != id) {
          this.update()
          this.thisMusicIndex = id > this.musicList.length - 1 ? 0 : id;
          this._getInfo()
        }
      },
      //去除特殊符号,暂未发现作用
      // Ltrim(s) {
      //   return s.replace(/(^\s*)/g, "");
      // },
      // Rtrim(s) {
      //   return s.replace(/(\s*$)/g, "");
      // },
      Player() {
        let self = this;
        let player = $('#music')[0]
        let playerTimer = setInterval(timer, 1000)

        function timer() {
          self.currentProgress = `${(player.currentTime / player.duration) * 100}%`
          //接着这里写歌词滚动
          if (player.currentTime >= self.wordTime[self.o + 1]) {
            self.top += Number.parseInt($('.music_word').eq(self.o).height() + Number.parseInt($('.music_word').eq(self.o).css('marginTop')));
            if (self.top >= $('.music_words').height() / 2 - 11) {  //开始滚动的高度
              self.wordsTop += -Number.parseInt($('.music_word').eq(self.o).height() + Number.parseInt($('.music_word').eq(self.o).css('marginTop')));
            }
            self.wordIndex = self.o + 1;
            self.o++;
          }
        }

        //进度条控制
        $('.progress_b').on('mousedown', (e) => {
          let pro = ((e.clientX - $('.progress_b').offset().left) / $('.progress_b').width())
          clearInterval(playerTimer);
          this.currentProgress = `${pro * 100}%`

          $(document).on('mousemove', (e) => {
            pro = ((e.clientX - $('.progress_b').offset().left) / $('.progress_b').width())
            this.currentProgress = `${pro * 100}%`
          })

          $(document).on('mouseup', () => {
            player.currentTime = player.duration * pro;
            let c_arr = [...this.wordTime];
            c_arr.push(player.currentTime);
            c_arr.sort((n1, n2) => n1 - n2)
            let now_o = c_arr.indexOf(player.currentTime) - 1;
            let diff_h = 0;
            if (this.o < now_o) {
              for (let i = this.o; i < now_o; i++) {
                diff_h += -Number.parseInt($('.music_word').eq(i).height() + Number.parseInt($('.music_word').eq(i).css('marginTop')))
              }
            } else {
              for (let i = now_o; i < this.o; i++) {
                diff_h += Number.parseInt($('.music_word').eq(i).height() + Number.parseInt($('.music_word').eq(i).css('marginTop')))
              }
            }
            this.wordsTop += diff_h;
            self.wordIndex = this.o = now_o;
            console.log(self.wordIndex)
            clearInterval(playerTimer);
            playerTimer = setInterval(timer, 1000);
            $(document).unbind('mousemove');
            $(document).unbind('mouseup');
          })
        })
        //播放暂停按钮控制
        $('.control_icon').on('click', () => {
          if (player.paused) {
            player.play();
            this.status = true
          } else {
            player.pause()
            this.status = false
          }
        })
      },
      update() {
        this.top = 0
        this.o = 0
        this.wordIndex = 0
        this.wordsTop = 0
        this.currentProgress = '0%'
      }
    }
  }
</script>
<style scoped>
  /*弹窗内容过渡效果*/
  .music_alert-enter, .music_alert-leave-to {
    transform: translateY(-30px);
    opacity: 0;
  }

  /*弹窗样式*/
  .music_alert {
    position: absolute;
    top: 30px;
    left: 0px;
    right: 0px;
    margin: auto;
    padding: 10px;
    padding-top: 5px;
    padding-bottom: 5px;
    width: 200px;
    height: auto;
    border-radius: 2px;
    background-color: white;
    text-align: center;
    transition: all .5s;
    color: black;
    font-size: 12px;
  }

  /*搜索内容过渡效果*/
  .music_search-enter-active,
  .music_search-enter, .music_search-leave-to {
    opacity: 0;
  }

  .list_box {
    width: 100%;
    height: 100%;
    position: fixed;
    top: 0;
    left: 0;
    background: -webkit-radial-gradient(center, ellipse cover, #353138 0%, #1a181c 100%);
    z-index: 499;
    transition: all .5s;
  }

  /*退出按钮*/
  .list_close {
    position: absolute;
    right: 20px;
    top: 10px;
    width: 40px;
    height: 40px;
    text-align: center;
    line-height: 40px;
    color: rgba(225, 225, 225, 0.9);
    cursor: pointer;
    font-size: 20px;
  }

  /*左边*/
  .list_l {
    width: 950px;
    height: 100%;
    float: left;
    padding: 15px 50px;
    margin-left: 120px;
    box-sizing: border-box;
    position: relative;
  }

  .music_list {
    width: 1320px;
    height: 840px;
    position: absolute;
    left: 0px;
    top: 0px;
    margin: auto;
    border-radius: 5px;
  }

  /*歌曲菜单*/
  .music_type {
    width: 120px;
    height: auto;
    overflow: hidden;
    list-style: none;
    position: absolute;
    left: -120px;
    top: 90px;
  }

  .music_type li {
    width: 100%;
    height: 35px;
    border: 1px solid rgba(225, 225, 225, 0.6);
    color: rgba(225, 225, 225, 0.6);
    line-height: 35px;
    text-align: center;
    box-sizing: border-box;
    border-radius: 3px;
    margin-top: 20px;
    cursor: pointer;
    transition: all .5s;
    font-size: 12px;
  }

  .music_type li:first-child {
    margin-top: 0px;
  }

  .music_type li:hover {
    border: 1px solid rgba(225, 225, 225, 1);
    color: rgba(225, 225, 225, 1);
  }

  /*菜单点击样式*/
  .type_active {
    border: 1px solid rgba(225, 225, 225, 1) !important;
    color: rgba(225, 225, 225, 1) !important;
  }

  /*搜索栏*/

  .list_title {
    width: 100%;
    height: 40px;
    color: white;
    position: relative;

  }

  .music_search_box {
    position: absolute;
    right: 0px;
    top: 0px;
    width: 250px;
    height: 25px;
    z-index: 500;
  }

  /*搜索框*/
  .music_search {
    width: 100%;
    height: 100%;
    border: 0px;
    outline: none;
    background-color: rgba(225, 225, 225, 0.9);
    border-radius: 2px;
    padding-left: 10px;
    padding-right: 10px;
    font-size: 12px;
  }

  /* 循环点击按钮 */
  .music_state {
    width: 15px;
    margin-left: 40px;
    cursor: pointer;
  }

  .search_list {
    width: 100%;
    height: auto;
    overflow: hidden;
    position: absolute;
    top: 28px;
    background-color: rgba(255, 255, 255, 0.9);
    border-radius: 2px;
    transition: all .5s;
  }

  .search_list li {
    width: 100%;
    height: 25px;
    line-height: 25px;
    color: rgba(26, 24, 28, 0.8);
    padding-left: 10px;
    padding-right: 10px;
    cursor: pointer;
    transition: all .5s;
    font-size: 12px;
  }

  .search_list li:hover {
    background-color: rgba(200, 200, 200, .9)
  }

  /*歌曲列表*/
  ._list {
    width: 100%;
    height: 620px;
    list-style: none;
    margin-left: -40px;
  }

  ._list li {
    width: 100%;
    height: 60px;
    line-height: 60px;
    border-bottom: 1px solid rgba(150, 150, 150, 0.1);
    transition: all .5s;
    position: relative;
  }

  ._list li:hover {
    background-color: rgba(26, 24, 28, 0.3);
  }

  ._list li span, .music_ul_title span {
    display: block;
    width: 200px;
    height: 100%;
    margin-left: 20px;
    color: rgba(225, 225, 225, 0.8);
    font-size: 12px;
    float: left;
    overflow: hidden;
  }

  /*播放暂停按钮*/
  .music_button {
    width: 150px;
    height: 60px;
    position: absolute;
    right: 0px;
    transition: all .5s;
  }

  .list_play {
    width: 28px;
    height: 28px;
    background-size: 100% 100%;
    margin-top: 16px;
    cursor: pointer;
    opacity: 0.5;
    transition: all .5s;
    float: left;
    margin-left: 10px;
  }

  .list_play:first-child {
    margin-left: 0px;
  }

  .list_play:hover {
    opacity: 1;
  }

  /*歌曲歌手专辑*/
  .music_ul_title {
    width: 100%;
    height: 40px;
    line-height: 40px;
    border-radius: 2px;
    border: 0px;
  }

  /*翻页按钮*/
  .list_page {
    width: 100px;
    height: 40px;
    position: relative;
    top: -30px;
    left: 40%;
  }

  .page_last {
    position: absolute;
    left: 0;
    width: 35px;
    height: 35px;
    line-height: 35px;
    text-align: center;
    font-size: 35px;
    cursor: pointer;
    color: white;
    -moz-user-select: none; /*火狐*/
    -webkit-user-select: none; /*webkit浏览器*/
    -ms-user-select: none; /*IE10*/
    user-select: none;
  }

  .page_next {
    position: absolute;
    right: 0;
    width: 35px;
    height: 35px;
    line-height: 35px;
    text-align: center;
    font-size: 35px;
    cursor: pointer;
    color: white;
    -moz-user-select: none; /*火狐*/
    -webkit-user-select: none; /*webkit浏览器*/
    -ms-user-select: none; /*IE10*/
    user-select: none;
  }

  .page_last:hover, .page_next:hover, .list_close:hover {
    background: rgba(171, 205, 189, .5);
  }

</style>
<style scoped>
  /* 自动播放音乐 */
  #music {
    width: 0px;
    height: 0px;
  }

  /*播放器方块*/
  .boxx {
    width: 100px;
    height: 100px;
    border-radius: 5px;
    position: fixed;
    z-index: 500;
    bottom: 2.5rem;
    right: 4rem;
    transition: all 0.5s;
    transform: scale(0.5);
  }

  .boxx_active { /*点击后的样式*/
    width: 300px;
    height: 300px;
    transform: scale(1);
  }

  /* 播放器圈圈 */
  .pan {
    width: 120px;
    height: 120px;
    background-size: 100% 100%;
    position: absolute;
    left: 210px;
    bottom: 300px;
    padding: 16px;
    animation: pan 10s linear infinite;
    z-index: 103;
    transition: all .5s;
    cursor: pointer;
    border-radius: 50%;
  }

  .pan:hover {
    box-shadow: 0px 0px 10px rgb(100, 100, 100);
    opacity: 1;
    left: 138px;
  }

  .pan_active { /*点击后*/
    width: 40px;
    height: 40px;
    background-size: 100% 100%;
    left: 10px;
    top: 23px;
    padding: 6px;
    opacity: 0.8;
    z-index: 103;
  }

  .pan_active:hover {
    left: 10px
  }

  .pan_c {
    width: 100%;
    height: 100%;
    border-radius: 50%;
  }

  /* 方塊里的內容 */
  .box {
    width: 100%;
    height: 100%;
    overflow: hidden;
    background-size: 100% 100%;
    border-radius: 5px;
    transition: all .5s;
    opacity: 0;
  }

  .box_active {
    width: 100%;
    height: 100%;
    background-size: 100% 100%;
    border-radius: 5px;
    overflow: unset;
    /* older safari/Chrome browsers */
    -webkit-opacity: 1;
    /* Netscape and Older than Firefox 0.9 */
    -moz-opacity: 1;
    /* Safari 1.x (pre WebKit!) 老式khtml内核的Safari浏览器*/
    -khtml-opacity: 1;
    /* IE9 + etc...modern browsers */
    opacity: 1;
    /* IE 4-9 */
    filter: alpha(opacity=100);
    /*This works in IE 8 & 9 too*/
    -ms-filter: "progid:DXImageTransform.Microsoft.Alpha(Opacity=100)";
    /*IE4-IE9*/
    filter: progid:DXImageTransform.Microsoft.Alpha(Opacity=100);
  }

  /* 背景模糊 */
  .music_1 {
    width: 100%;
    height: 100%;
    position: absolute;
    -webkit-filter: blur(10px);
    filter: blur(10px);
    background-size: 100% 100%;
    transition: all .5s;
    z-index: 99;
    opacity: 0;
  }

  .music_2 {
    width: 100%;
    height: 100%;
    position: absolute;
    filter: blur(5px);
    background-size: 100% 100%;
    transition: all .5s;
    opacity: 0;
  }

  .music_3 {
    width: 100%;
    height: 100%;
    position: absolute;
    background-color: rgba(0, 0, 0, 0.1);
    transition: all .5s;
    border-radius: 5px;
    filter: blur(2px);
  }

  .shlter_active {
    opacity: 1;
    z-index: auto;
  }

  /* 歌曲播放部分 */
  .music_dis {
    width: 100%;
    height: 240px;
    position: relative;
  }

  /*三点*/
  .dis_list {
    position: absolute;
    right: 10px;
    top: 10px;
    width: 40px;
    height: 30px;
    line-height: 30px;
    text-align: center;
    cursor: pointer;
    font-size: 12px;
    color: white;
  }

  .music_title {
    width: 100%;
    height: 40px;
    margin: 0;
    padding: 0;
    text-align: center;
    color: white;
    line-height: 60px;
    font-size: 15px;
  }

  .music_intro {
    width: 100%;
    text-align: center;
    color: rgba(255, 255, 255, 0.6);
    line-height: 30px;
    font-size: 12px;
    margin: 0;
    padding: 0;
  }

  .music_words {
    width: 240px;
    height: 160px;
    color: rgba(255, 255, 255, 0.6);
    text-align: center;
    overflow: hidden;
    position: relative;

  }

  .music_words_box {
    width: 100%;
    height: auto;
    position: absolute;
    top: 0;
    left: 25px;
    transition: all .5s;
    text-align: center;
  }

  .music_words li {
    list-style: none;
    font-size: 12px;
    margin: 5px 0 0 -20px;
    min-height: 18px;
    text-align: center;
  }

  .word_highlight {
    color: white;
  }

  .control_box {
    width: 90%;
    height: 60px;
    position: relative;
    margin: auto;
  }

  .control_button {
    width: 14px;
    height: 14px;
    position: absolute;
    left: 20px;
    bottom: 15px;
    user-select: none;

  }

  .control_icon {
    height: 100%;
    cursor: pointer;
  }

  .progress_b {
    width: 150px;
    height: 4px;
    border-radius: 10px;
    background-color: rgb(194, 194, 196);
    cursor: pointer;
    position: absolute;
    margin: auto;
    left: 0;
    right: 0;
    bottom: 20px;
  }

  .progress_c {
    width: 50%;
    height: 100%;
    background-color: rgb(232, 60, 60);
    position: relative;
    border-radius: 10px;
    max-width: 100%;
  }

  .progress_circle {
    position: absolute;
    width: 15px;
    height: 15px;
    right: -7.5px;
    top: -5.5px;
    background-color: white;
    border: 1px solid rgb(220, 220, 220);
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .progress_circle_c {
    width: 7px;
    height: 7px;
    background: #abcdef;
    border-radius: 50%;

  }

</style>