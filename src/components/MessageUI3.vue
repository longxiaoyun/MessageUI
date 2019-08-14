<template>
<!-- 参考 https://zhuanlan.zhihu.com/p/32878390-->
<!--  https://segmentfault.com/q/1010000012001151-->
<!--  https://www.w3schools.com/howto/howto_css_chat.asp-->
  <div id="message-container" v-drag>
    <!--  the start  -->
    <section class="theme-color header-container">
       <div class="slider-toggle">
         <button class="theme-color tog-btn"  @click="toggleSlider">切换</button>
       </div>
       <p class="header-title">{{ this.chatTitle }}</p>
    </section>
    <div :class="isCollapse ? 'layout' : 'layout-hide'">
      <section class="slider-common" :class="isCollapse ? 'slider-container' : 'slider-hide'">
        <ul :class="isCollapse ? 'ul-line' : 'ul-line-big'">
          <li v-for="item in concatList">
            <h2 v-if="isCollapse" class="concat-title">{{ item.title }}</h2>
            <ul :class="isCollapse ? 'ul-line': 'ul-line-big'" v-for="subitem in item.contacts">
              <li v-if="isCollapse" @click="getItem(subitem)">
                <img class="user-header" alt="头像" :src="subitem.avatar"/>
                <span class="concat-name">{{ subitem.name }}</span>
              </li>
              <li v-else @click="getItem(subitem)">
                <img class="user-header-big" alt="头像" :src="subitem.avatar"/>
              </li>
            </ul>
          </li>
        </ul>
      </section>

      <section :class="isCollapse ? 'main-container' : 'main-hide'">
        <ul class="chat-box">
          <template v-for="item in messageRecords">
            <div class="chat-container" v-if="item.type === 1">
              <img :src="item.fromAvatar" alt="Avatar" style="width:100%;">
              <p class="f-c">{{ item.content }}</p>
              <span class="time-right">11:00</span>
            </div>

            <div class="chat-container darker" v-if="item.type === 2">
              <img :src="item.fromAvatar" alt="Avatar" class="right" style="width:100%;">
              <p class="f-c">{{ item.content }}</p>
              <span class="time-left">11:01</span>
            </div>

          </template>
        </ul>
      </section>
      <section class="theme-color footer-container">
        <div class="send-container">

          <textarea ref="text" v-model="message" @keyup="onKeyup" @click="showEmoji=false"></textarea>
          <div class="send" @click="sendMsg">
            <span>发送(ent)</span>
          </div>
        </div>
      </section>
    </div>

    <!--  the end  -->
  </div>
</template>

<script>

export default {
  name: "MessageUI",
  props: {
    messageRecords: {
      type: Array,
      default: () => []
    },
    concatList: {
      type: Array,
      default: () => []
    }
  },
  data() {
    return {
      z_index: 12,
      isCollapse: true,
      message: '',
      chatTitle: '大壮'
    }
  },
  directives: {
    drag: {
      bind: (el, binding) => {
        //绑定默认样式
        el.style.zIndex = this.z_index;

        //如果为编辑状态
        if (binding.value || binding.value === undefined) {
          //定义该元素的 top left width height
          let x, y, w, h;
          //鼠标的起始和结束坐标
          let cx_start, cy_start, cx_end, cy_end;
          //判断鼠标样式
          el.onmousemove = e => {
            //获取鼠标当前位置
            let cx_now = e.clientX;
            let cy_now = e.clientY;
            //获取div右下角相对浏览器的位置
            let {
              top: el_top,
              left: el_left,
              width: el_width,
              height: el_height
            } = el.getBoundingClientRect();
            let el_bottom_height = el_top + el_height;
            let el_right_width = el_left + el_width;
            //判断鼠标是否在div下边界
            let mouse_in_bottom =
              cy_now <= el_bottom_height + 5 && cy_now >= el_bottom_height - 5;
            //判断鼠标是否在div右边界
            let mouse_in_right =
              cx_now <= el_right_width + 5 && cx_now >= el_right_width - 5;
            if (mouse_in_bottom && mouse_in_right) {
              el.style.cursor = "se-resize";
            } else if (mouse_in_right) {
              el.style.cursor = "e-resize";
            } else if (mouse_in_bottom) {
              el.style.cursor = "s-resize";
            } else {
              el.style.cursor = "move";
            }
          };
          el.onmousedown = e => {
            let mouse = el.style.cursor;
            //更改默认样式
            // el.style.backgroundColor = "rgba(0,0,0,.5)";
            el.style.zIndex = this.z_index+1;
            //对象解构赋值
            let { left: el_x, top: el_y, width: el_w, height: el_h } = window.getComputedStyle(el);
            x = el_x;
            y = el_y;
            w = el_w;
            h = el_h;
            console.log(x,y,w,h)
            cx_start = e.clientX;
            cy_start = e.clientY;
            //绑定移动事件
            document.onmousemove = e => {
              cx_end = e.clientX;
              cy_end = e.clientY;
              //默认左下方向配置
              let x_move = cx_end - cx_start;
              let y_move = cy_end - cy_start;
              let direct = ["width", "height"];
              let pos = [w, h];
              let move = [x_move, y_move];
              let limit = 50;
              //判断鼠标的类型进行对应的操作
              switch (mouse) {
                case "e-resize":
                  direct = ["width"];
                  pos = [w];
                  move = [x_move];
                  break;
                case "s-resize":
                  direct = ["height"];
                  pos = [h];
                  move = [y_move];
                  break;
                case "move":
                  direct = ["left", "top"];
                  pos = [x, y];
                  limit = 0;
                  break;
              }
              handle_div(direct, pos, move, limit);
            };
            //取消移动事件
            document.onmouseup = e => {
              //还原默认样式
              el.style.zIndex = this.z_index;
              // el.style.backgroundColor = "rgba(0,0,0,1)";
              document.onmousemove = null;
            };
            /**
             * 操作DOM位置和大小方法
             * @param direct 方向
             * @param pos 尺寸/坐标
             * @param move 拖动距离
             * @param limit 限定范围
             */
            function handle_div(direct, pos, move, limit) {
              for (let i = 0; i < direct.length; i++) {
                let val = parseInt(pos[i]) + move[i];
                val = val <= limit ? limit : val;
                el.style[direct[i]] = val + "px";
              }
            }
          };
        } else {
          el.style.cursor = "default";
          //移除点击事件
          el.onmousedown = null;
          el.onmousemove = null;
        }
        // the end
      }
    }
  },
  methods: {
    toggleSlider() {
      this.isCollapse=!this.isCollapse
    },
    onKeyup (e) {
      if ( e.keyCode === 13 ) {
        this.sendMsg()
      }
    },
    sendMsg() {
      if (this.message.length === 0) {
        console.log("不能为空")
        return
      }
      this.messageRecords.push({
        type: 1,
        time: '2019-08-01 10:22:12',
        fromName: '游客',
        toName:'张三',
        content: this.message,
        fromAvatar: 'data:image/png;base64,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'
      })
      this.message='';
      this.scrollToBottom();
    },
    scrollToBottom:function() {
      this.$nextTick(() => {
        const chatbox = document.getElementsByClassName('main-container')[0]
        chatbox.scrollTop = chatbox.scrollHeight;
      })

    },
    getItem(subitem) {
      this.chatTitle=subitem.name
    },
  },
  watch: {
    message() {

    }
  },
}
</script>

<style scoped>
#message-container{
  position: relative;
  top: 10px;
  left: 10px;
  width: 500px;
  height: 400px;
  box-shadow: grey 10px 0px 10px 0px;
}


.layout {
  display: flex;
  flex-flow: wrap;
  max-height: 100%;
  height: 80%;
}
.header-container {
  height: 50px;
  display: flex;
  flex-flow: wrap;
}
/*header样式 start*/
.slider-toggle{
  flex: 0 0 30%;
  margin-left: calc((100% - 3 * 30%) / 4);
  margin-top: 10px;
}

.header-title {
  flex: 0 0 30%;
  margin-left: calc((100% - 3 * 30%) / 4);
  text-align: center;
}
/*  end */


.slider-container{
  flex: 0 0 30%;
  margin-top: 24px;
  margin-left: calc((100% - 3 * 30%) / 4);
}
.main-container{
  flex: auto;
  margin-left: calc((100% - 3 * 30%) / 4);
  overflow-y: scroll;
  max-height: 100%;
  max-width: 100%;
  overflow-x: hidden;
  padding-right: 10px;
}

.slider-common {
  background: white;
  margin-left: 0;
  margin-top: 0;
  border-right-style: groove;
  border-left-style: groove;

}
/*  the end */

/* 收起菜单栏样式 start */
.layout-hide {
  display: flex;
  flex-flow: wrap;
  max-height: 100%;
  height: 80%;
}
.slider-hide {
  flex: 0 0 10%;
}
.main-hide {
  flex: 0 0 80%;
  margin-left: calc((100% - 3 * 30%) / 4);
}
/* 收起菜单栏样式 end */


/* 头像图片 */
.user-header {
  width: 1em;
  height: 1em;
  border-radius:1em;
}
.user-header-big {
  width:2em;
  height: 2em;
  border-radius: 2em;
  padding-right: 1em;
}
/* 头像图片 end */


/* 字体大小等 */
.concat-title {
  font-size: 0.75em;
}
.concat-name {
  font-size: 0.5em;
}
.ul-line {
  margin-left:1.2em;
  padding-left: 0.2em;
  list-style: none;
  cursor: pointer;
}
.ul-line-big {
  margin-left:0.2em;
  padding-left: 0.2em;
  list-style: none;
  cursor: pointer;
}





/* 聊天界面 start */
.chat-box {
  height: 100%;
  background: white;
  list-style: none;
  padding-left: 0;
  padding-right: 2em;
}


.chat-container {
  border: 2px solid #dedede;
  background-color: #f1f1f1;
  border-radius: 5px;
  padding: 10px;
  margin: 10px 0;
  margin-bottom: 50px;
}

.darker {
  border-color: #ccc;
  background-color: #ddd;
}

.chat-container::after {
  content: "";
  clear: both;
  display: table;
}

.chat-container img {
  float: left;
  max-width: 40px;
  width: 100%;
  margin-right: 20px;
  border-radius: 50%;
}

.chat-container img.right {
  float: right;
  margin-left: 20px;
  margin-right:0;
}

.time-right {
  float: right;
  color: #aaa;
  font-size: 0.75em;
}

.time-left {
  float: left;
  color: #999;
  font-size: 0.75em;
}
.f-c {
  font-size: 0.85em;
}

/*  聊天界面 end*/

/*  页脚 */
.footer-container {
  height: 70px;
  position: absolute;
  bottom: 0;
  width: 100%;
}
.send-container {
  background: white;
  margin-top: 0.5em;
  margin-left: 0.5em;
  margin-right: 0.5em;
  height: 3.5em;
}

textarea {
  box-sizing: border-box;
  padding: 0.5em 1em;
  width: 100%;
  border: none;
  outline: none;
  font-family: "Micrsofot Yahei";
  resize: none;
  height: 100%;
  font-size: 16px
}
.send {
  position: absolute;
  bottom: 1.5em;
  right: 1em;
  width: 75px;
  height: 28px;
  line-height: 28px;
  box-sizing: border-box;
  text-align: center;
  border: 1px solid #e5e5e5;
  border-radius: 3px;
  background: #f5f5f5;
  font-size: 14px;
  color: #7c7c7c;

}
.send:hover {
  background: rgb(18,150,17);
  color: #fff;
  cursor: pointer;
}

/* 页脚 the end */


.tog-btn {
  width: 45px;
  height: 32px;
  border-width: 0px;
  border-radius: 3px;
  cursor: pointer;
  outline: none;
  font-family: Microsoft YaHei;
  color: white;
  font-size: 10px;
}

.theme-color {
  background: #4E8CFF;
}
</style>


