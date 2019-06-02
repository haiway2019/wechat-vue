<template>
  <transition name="slide">
    <div class="chatroom">
      <div class="back">
        <router-link to='/chat' >
          <img class="img" src="../../assets/返回.png" @click='clearContent' height="16" width="19" />
        </router-link>
        <span class="dissname">{{this.info.dissname}}</span>
        <span class="logo" @click="gotoUser(info)">
          <img src="../../assets/我.png" height="28" width="28">
          <!-- <span class="icon-user"></span> -->
        </span>
      </div>
      <div class="content">
        <div class="content-wrapper" ref="wrapper">
          <div class="content-text">
            <div class="content-top">
              <p>————有我小诸葛在，无所不知，请尽管问————</p>
            </div>
            <div class="content-body" ref="body">
              <ul class="inHtml" v-for="item in content">
                <li class="ask"  v-show="item.askContent">
                  <img :src="item.askImg" />
                  <p>{{item.askContent}}</p>
                </li>
                <li class="reply" v-show="item.replyContent">
                  <img :src="item.replyImg" />
                  <p v-html="item.replyContent"></p>
                </li>
              </ul>
            </div>
          </div>
        </div>
      </div>
      <div class="bottom">
        <div class="send">
          <input
            type="text"
            placeholder="请输入聊天内容"
            class="sText"
            ref="sTest"
          />
          <input type="button" class="btn" value="发送" @click="sendContent" />
        </div>
      </div>
      <router-view></router-view>
    </div>
  </transition>
</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll'
  import {mapGetters} from 'vuex'
  import SockJS from  'sockjs-client';
  import Stomp from 'stompjs';

  export default {
    components: {
      BScroll
    },
    data () {
      return {
        text: '', // 输入框的文字
        stompClient: '',
        content: [
        ]
      }
    },
    computed: {
      ...mapGetters([
        'info'
      ])
    },
    created () {

    },
    mounted () {
      this.$nextTick(() => {
        this.scroll = new BScroll(this.$refs.wrapper, {
          click: true
        })
      });
      this.connection();
    },
    beforeDestroy () {
      this.disconnect()
    },
    methods: {
      back () {
        sessionStorage.removeItem("faqSessionId");
        this.$router.back()   // 返回上一级
      },
      gotoUser (info) {
        sessionStorage.removeItem("faqSessionId");
        this.$router.push({
          path: `/chatroom/user`
        });
      },
      sendContent () {
        this.text = this.$refs.sTest.value
        if (this.text !== '') {
            this.content.push({
              askImg: require('../../assets/me/minion.png'),
              askContent: this.text
            });
            console.log("sessionId:"+sessionStorage.getItem("faqSessionId"));
            this.stompClient.send("/v1/faq",
              {},
              JSON.stringify({"faqSessionId":sessionStorage.getItem("faqSessionId"),"name": "haiway" ,"message":this.text}),
            );

          // this.$http.get('http://localhost:8080/test/ask',{params: {content:this.text}}).then(response => {
          //   this.content.push({
          //     askImg: require('../../assets/me/minion.png'),
          //     askContent: this.text
          //   })

          //   this.content.push({
          //     replyImg: this.info.imgurl,
          //     replyContent: response.bodyText
          //   })
          // }, response => {
          //   console.log('error')
          // })
        }
        this.$refs.sTest.value = '' // 清空输入框的内容
      },
      clearContent () {
        this.content = []
      },
      connection () {
        // 建立连接对象
        let socket = new SockJS('http://localhost:8080/ws');
        // 获取STOMP子协议的客户端对象
        this.stompClient = Stomp.over(socket);
        // 向服务器发起websocket连接
        this.stompClient.connect({},() => {
          this.stompClient.subscribe('/topic/callback', (msg) => { // 订阅服务端提供的某个topic
                console.log('广播成功')
                // console.log(msg);  // msg.body存放的是服务端发送给我们的信息

              let dataKS = JSON.parse(msg.body);
              sessionStorage.setItem("faqSessionId",dataKS.faqSessionId);

                this.content.push({
                  replyImg: this.info.imgurl,
                  replyContent: dataKS.message
                })

            });
        }, (err) => {
            // 连接发生错误时的处理函数
            console.log('失败')
            console.log(err);
        });
      },
      disconnect () {
        if(this.stompClient){
          this.stompClient.disconnect();
          sessionStorage.removeItem("faqSessionId");
        }
      }
    }
  }
</script>

<style scoped>
  .chatroom{
    position: fixed;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    z-index: 19;
    background-color: #ebebeb;
  }
  .back{
    background: #1e2b39;
    height: 50px;
    color: #fff;
    position: fixed;
    width: 100%;
  }
  .back .img{
    position: absolute;
    top: 25px;
    margin-top: -8px;
    left: 14px;
  }
  .back .dissname{
    position: absolute;
    font-size: 20px;
    top: 25px;
    margin-top: -10px;
    left: 50px;
    padding-left: 10px;
    border-left: 1px solid #000;
  }
  .back .logo{
    position: absolute;
    font-size: 20px;
    top: 30px;
    margin-top: -15px;
    right: 20px;
  }
  .content{
    position: fixed;
    top: 50px;
    bottom: 50px;
    left: 0;
    right: 0;
    /*border: 1px solid red;*/
  }
  .content-wrapper{
    height: 100%;
    overflow: hidden;
  }
  .content-top{
    font-size: 14px;
    color: rgba(153,153,153,0.6);
    text-align: center;
    margin-top: 4px;
  }
  .content-body{
    position: relative;
    padding: 20px 10px;
    /*overflow: hidden;*/
    /*border: 1px solid blue;*/
  }
  .content-body li {
    position: relative;
    overflow: hidden;
    margin-bottom: 15px;
    line-height: 28px;
  }
  .inHtml img {
    position: relative;
    width: 30px;
    height: 30px;
  }
  .ask {
    text-align: right;
  }
  .reply {
    text-align: left;
  }
  .ask img {
    float: right;
    margin-left: 15px;
  }
  .reply img {
    float: left;
    margin-right: 15px;
  }
  .reply p, .ask p {
    border-radius: 4px;
    text-align: left;
    font: 14px 'Microsoft YaHei';
    line-height: 30px;
  }
  .ask p {
    float: right;
    padding: 3px 10px;
    max-width: 182px;
    background: #228b22;
    color: #fff;
  }
  .reply p {
    left: 2pc;
    float: left;
    padding: 3px 10px;
    max-width: 190px;
    background: #fff;
  }
  .bottom{
    position: fixed;
    height: 50px;
    bottom: 0;
    left: 0;
    right: 0;
    background-color: #fff;
  }
  .send{
    display: flex;
  }
  .sText{
    flex: 6;
    height: 30px;
    margin: 10px;
    border: 0;
    padding-left: 8px;
    border-bottom: 1px solid rgba(153,153,153,0.8);
    /*border: 1px solid rgba(153,153,153,0.8);*/
    font-size: 15px;
  }
  .sText.active{
    background-color: red;
  }
  .btn{
    flex: 1;
    width: 65px;
    height: 30px;
    margin: 10px 10px;
    border: 0;
    border-radius: 5px;
    /*float: right;*/
    text-align: center;
    font-size: 18px;
    color: white;
    background-color: #09BB07;
  }

  .slide-enter-active,.slide-leave-active{
    transition: all 0.3s;
  }
  .slide-enter,.slide-leave-to{
    transform: translate3d(100%, 0, 0);
  }
</style>
