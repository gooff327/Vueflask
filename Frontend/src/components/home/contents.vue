<!--suppress ALL -->
<template>
  <transition appear>
    <div>
      <top-headers class="headers"></top-headers>
      <div ref="wrapper" class="wrapper">
        <div class="contentWrapper">
          <el-card shadow="hover" :body-style="{padding: '6px' }" class="item" v-for="(key,item) in contents"
                   :key="item">
            <div class="headbar">
              <img @click="showUserDetails(key.author)" class="avatar" :src="key.uavatar"
                   alt="">
              <span class="username">{{key.author}}</span>
              <i class="el-icon-more-outline"></i>
            </div>
            <div class="contentImage">
              <img onmouseover="displayDesc" class="innerPic" :src="key.img" :preview="key.pid"
                   alt="">
              <span class="imageDesc" v-if="key.desc">{{key.desc}}</span>
            </div>
            <div class="bottom">
              <span class="bottomText">{{key.date.slice(4,-3)}}发布</span>
              <span class="botttomIcon">
              <i style="color: #EE4957" @click="likeEvent(key)" v-if="admire[key.pid] ===true" class="fa fa-heart"
                 aria-hidden="true"></i>
              <i @click="likeEvent(key)" v-else class="fa fa-heart-o" aria-hidden="true"></i>
              <i @click="commentEvent(key.pid,key.uid)" class="fa fa-comment-o" aria-hidden="true"></i>
              <i @click="forwardEvent(key)" class="fa fa-paper-plane-o" aria-hidden="true"></i>
           </span>
            </div>
          </el-card>
        </div>
      </div>
    </div>
  </transition>
</template>

<script>
  import comment from '../../components/Common/comment'
  import BScroll from 'better-scroll'
  import store from '../../vuex/user'
  import header from '../../components/header/homeHeader'

  export default {
    data () {
      return {
        showContents: true,
        dropDown: false,
        updatePanel: false,
        emptyContent: true,
        refreshData: {},
        scrollpos: 0,
        admire: {}
      }
    },
    components: {
      'top-headers': header,
      'comment': comment
    },
    created: function () {
      this.getAdmireList()
    },
    watch: {
      $route (to, from) {
        this.getAdmireList()
        if (from.path === '/following') {
          let position = this.scroll.y
          store.commit('changefPosition', position)
        }
        if (to.path === '/following') {
          this.scroll.refresh()
          this.scroll.scrollTo(0, store.state.positions.fPosition)
        }
      }
    },
    mounted () {
      this.$nextTick(() => {
        this.initHomeScroll()
      })
    },
    methods: {
      initHomeScroll: function () {
        if (!this.scroll) {
          this.scroll = new BScroll(this.$refs.wrapper, {
            click: true,
            bounce: true,
            scrollY: true,
            probeType: 3,
            pullUpLoad: {
              threshold: 0
            },
            pullDownRefresh: {
              threshold: 20
            }
          })
          this.scroll.on('scroll', (position) => {
            if (!this.scroll.isInTransition && !this.scroll.isAnimating && position.y > 20) {
              this.dropDown = true
            } else {
              this.dropDown = false
            }
          })

          this.scroll.on('pullingDown', () => {
            this.$emit('refresh')
            console.log('刷新数据')
            setTimeout(() => {
              this.scroll.finishPullDown()
              this.scroll.refresh()
            }, 200)
          })
          this.scroll.on('pullingUp', () => {
            this.$emit('loadMore')
            setTimeout(() => {
              this.scroll.finishPullUp()
              this.scroll.refresh()
            }, 200)
          })
        } else {
          this.scroll.refresh()
        }
        this.setHight()
      },
      getAdmireList: function () {
        var url = this.GLOBAL.BASE_URL + '/api/v1/admire'
        this.$axios.get(url).then(function (res) {
          console.log(res)
          if (res.data.code === 521) {
            this.admire = res.data.admire
          }
        }.bind(this))
      },
      showUserDetails: function (username) {
        this.GLOBAL.showLoading()
        this.$router.push(`/user/${username}`)
      },
      likeEvent: function (key) {
        this.admire[key.pid] === true ? this.admire[key.pid] = false : this.admire[key.pid] = true
        let url = this.GLOBAL.BASE_URL + '/api/v1/admire'
        let admireList = JSON.stringify(this.admire)
        let admireThis = JSON.stringify({pid: key.pid, uid: key.uid, result: this.admire[key.pid]})
        let data = {
          admireList: admireList,
          admireThis: admireThis
        }
        this.$axios.post(url, data, {headers: {'Content-Type': 'Application/json'}}).then(function (response) {
          console.log('admire', response)
          if (response.data.code === 520 && this.admire[key.pid] === true) {
          }
        }.bind(this))
      },
      commentEvent: function (pid, vid) {
        this.$router.push({name: 'comment', params: {pid: pid, vid: vid}})
      },
      forwardEvent: function (key) {
        this.$router.push({name: 'forward', params: {pid: key.pid, passage: key}})
      }
    },
    props: {
      contents: {
        type: Object
      }
    },
    name: 'contents'
  }
</script>
<style scoped>
  [v-cloak] {
    display: none;
  }

  .fadein-enter-active, .fadein-leave-active {
    transition: opacity .5s
  }

  .fadein-enter, .fadein-leave-to {
    opacity: 0
  }

  .headbar {
    display: inline-block;
    vertical-align: top;
    height: 20%;
    font-size: 20%;
    padding-bottom: 0.2rem;
    width: 100%;
    position: relative;
  }

  .headers {
    position: fixed;
    top: 0rem;
    z-index: 1;
  }

  .wrapper {
    position: absolute;
    z-index: 2;
    left: 0;
    top: 0px;
    overflow: hidden;
    max-height: 94vh;
  }

  .imageDesc {
    position: absolute;
    bottom: 0;
    color: white;
    background-color: rgba(0, 0, 0, 0.6);
    width: 96vw;
    text-align: center;
    max-height: 40%;
    font-size: 0.8rem;
    font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
    border-bottom-left-radius: 4px;
    border-bottom-right-radius: 4px;
    padding-top: 0.1rem;
  }

  .username {
    display: inline-block;
    max-width: 70%;
    font-family: "Helvetica Neue", Helvetica, "PingFang SC", "Hiragino Sans GB", "Microsoft YaHei", "微软雅黑", Arial, sans-serif;
    font-size: 0.7rem;
    padding-top: 0.4rem;
    padding-left: 0.4rem;
  }

  .avatar {
    display: inline-block;
    width: 1.8rem;
    height: 1.8rem;
    border-radius: 50%;
    float: left;
  }

  .el-icon-more-outline {
    display: inline-block;
    height: 20%;
    float: right;
    padding-top: 4%;
    padding-right: 6%;
  }

  .bottomText {
    font-size: 0.6rem;
    color: gray;
    margin: 0;
    padding: 0;
  }

  .botttom {
    vertical-align: top;
  }

  .botttomIcon {
    display: inline-block;
    float: right;
    padding-right: 0.4rem;
    margin-bottom: 0.4rem;
  }

  .fa {
    padding-right: 0.4rem;
    padding-left: 0.4rem;
    padding-bottom: 0.4rem;
  }

  .contentImage {
    margin-top: 0.2rem;
    /*min-width: 100%;*/
    /*max-height: 50%;*/
    width: 96vw;
    height: 54vw;
    position: relative;
  }

  .contentImage img {
    width: 100%;
    height: 100%;
    object-fit: cover;
  }

  .emptyContent {
    display: block;
    text-align: center;
    width: 100;
  }

  .innerPic {
    width: auto;
    height: auto;
    max-width: 100%;
    max-height: 100%;
    border-radius: 4px;
  }

  .fa-heart:hover {
    animation: admire 0.6s normal;
  }

  @keyframes admire {
    0% {
      transform: scale(1, 1)
    }
    20% {
      transform: scale(2.4, 2.4)
    }
    100% {
      transform: scale(1, 1)
    }
  }

  .fa-heart-o:hover {
    transition: color 1.5s;
  }

  .item {
    border-radius: 10px;
    margin-top: 2px;
    margin-bottom: 2px;
  }

  .moreInfo {
    display: inline-block;
    width: 100%;
    text-align: center;
    color: #34BE5B;
  }

  .contentWrapper {
    background-color: #cfcfc0;
    border-radius: 10px;
  }

  .el-card {
    margin-bottom: 3px;
  }
</style>
