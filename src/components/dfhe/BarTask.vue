<!--
 * @Author: your name
 * @Date: 2021-11-13 12:03:20
 * @LastEditTime: 2021-11-18 00:10:34
 * @LastEditors: Please set LastEditors
 * @Description: 打开koroFileHeader查看配置 进行设置: https://github.com/OBKoro1/koro1FileHeader/wiki/%E9%85%8D%E7%BD%AE
 * @FilePath: \win11_vue\src\components\HeDiFei\test.vue
-->
<template>
  <div class="taskBar" @click="taskbarClick">
    <div class="taskcont">
      <div class="tasksCont">
        <div
          class="taskIcon"
          v-for="appname in activeAppList"
          :key="appname"
          @mousedown="taskIconMouseDown"
          @mouseup="taskIconMouseUp"
          @mouseleave="taskIconMouseLeave"
          @click="taskIconClick(appname)"
          :id="appname + 'Task'"
        >
          <img
            :src="require('../../assets/img/appIcons/' + appname + '.png')"
            class=""
            alt=""
          />
        </div>
      </div>
      <div class="taskRight">
        <ul class="taskiconul" @click.stop="iSshowControlsCenter">
          <li class="taskiconli">
            <img :src="imgUrl.arrowupImg" alt="arrowup" />
          </li>
          <li class="taskiconli">
            <img :src="imgUrl.wifiImg" alt="wifi" />
          </li>
          <li class="taskiconli">
            <img :src="imgUrl.audio3Img" alt="audio" />
          </li>
          <li class="taskiconli">
            <img :src="imgUrl.batteryImg" alt="battery" />
          </li>
        </ul>
        <div class="taskDate">
          <ul class="dateDiv" @click.stop="showdateBox">
            <li>{{ today.date }}</li>
            <li>{{ today.time }}</li>
          </ul>
        </div>
      </div>
      <transition name="el-fade-in">
        <div v-show="dateBoxShow">
          <calendar></calendar>
        </div>
      </transition>
      <!-- <transition name="el-fade-in">
        <div v-show="controlCenterShow">
          <ControlCenter></ControlCenter>
        </div>
      </transition> -->
    </div>
  </div>
</template>

<script>
import { handleError } from '@vue/runtime-core';
import calendar from './BestCalendar.vue';
// import ControlCenter from './ControlCenter.vue';
export default {
  name: 'task-bar',
  components: { calendar },
  data() {
    return {
      dateBoxShow: false,
      controlCenterShow: false,
      timer: null,
      today: {
        date: '',
        time: '',
        dateBoxShow: false,
      },
      imgUrl: {
        wifiImg: require('../../assets/img/taskbarIcons/wifi.png'),
        batteryImg: require('../../assets/img/taskbarIcons/battery.png'),
        audio3Img: require('../../assets/img/taskbarIcons/audio3.png'),
        arrowupImg: require('../../assets/img/taskbarIcons/arrowup.png'),
      },

      smallerIcon: null,
      openedAppOrder: [],
      minApps: [],
      activeAppList: ['home', 'search', 'widget', 'settings', 'explorer'],
      initialAppList: ['home', 'search', 'widget', 'settings', 'explorer'],
    };
  },
  emits: ['showControls', 'responseTaskbarAction'],
  methods: {
    updateTime() {
      this.today = {
        date: new Date().toLocaleTimeString('en-US', {
          hour: 'numeric',
          minute: 'numeric',
        }),
        time: new Date().toLocaleDateString('en-US', {
          year: '2-digit',
          month: '2-digit',
          day: 'numeric',
        }),
      };
    },
    showdateBox() {
      this.dateBoxShow = !this.dateBoxShow;
      // 打开时间栏，关闭状态栏
      if(this.controlCenterShow) {
        this.controlCenterShow = false;
        this.$emit('showControls', this.controlCenterShow);
      }
    },
    iSshowControlsCenter() {
      this.controlCenterShow = !this.controlCenterShow;
      this.$emit('showControls', this.controlCenterShow);
      // 打开状态栏，关闭时间栏
      if(this.dateBoxShow) this.dateBoxShow = false;
    },

    // dssun
    taskbarClick() {
      this.closeDateAndControls();
    },
    closeDateAndControls() {
      if(this.controlCenterShow) {
        this.controlCenterShow = false;
        this.$emit('showControls', this.controlCenterShow);
      }
      if(this.dateBoxShow) this.dateBoxShow = false;
    },
    taskIconMouseDown(e) {
      if (e.target.nodeName.toLowerCase() === 'img')
        this.smallerIcon = e.target;
      else this.smallerIcon = e.target.firstChild;
      this.smallerIcon.classList.add('smaller');
    },
    taskIconMouseUp() {
      setTimeout(() => {
        if (!this.smallerIcon) return;
        this.smallerIcon.classList.remove('smaller');
        this.smallerIcon = null;
      }, 200);
    },
    taskIconMouseLeave() {
      setTimeout(() => {
        if (!this.smallerIcon) return;
        this.smallerIcon.classList.remove('smaller');
        this.smallerIcon = null;
      }, 200);
    },
    taskIconClick(appname) {
      this.$emit('responseTaskbarAction', appname);
    },
    changeApp(appname, e) {
      // e: 0 关闭 1 最小化 2 任务栏按下 3 打开 4 记事本打开txt文件特殊处理
      if (e === 0) {
        this.openedAppOrder.splice(this.openedAppOrder.indexOf(appname), 1);
        if (this.initialAppList.indexOf(appname) === -1) {
          this.activeAppList.splice(this.activeAppList.indexOf(appname), 1);
        }
      } else if (e === 1) {
        this.minApps.push(this.openedAppOrder.pop());
      } else if (e === 2) {
        // 任务栏按下
        if (this.minApps.indexOf(appname) !== -1) {
          // 如果最小化的应用列表有它，把它最大化
          this.openedAppOrder.push(appname);
          this.minApps.splice(this.minApps.indexOf(appname), 1);
        } else if (this.openedAppOrder.indexOf(appname) !== -1) {
          // 如果打开的应用中有它
          if(this.$emit("getActiveWin") === appname){
            // 如果它是当前窗口，最小化它
            this.openedAppOrder.splice(this.openedAppOrder.indexOf(appname), 1);
            this.minApps.push(appname);
          }
          else {
            // 否则，切换到它
            this.openedAppOrder.splice(this.openedAppOrder.indexOf(appname), 1);
            this.openedAppOrder.push(appname);
          }
          
        } else {
          // 否则，打开它
          this.openedAppOrder.push(appname);
        }
      } else {
        if (this.getAppState(appname)) {
          if (this.minApps.indexOf(appname) !== -1) {
            this.openedAppOrder.push(appname);
            this.minApps.splice(this.minApps.indexOf(appname), 1);
          }
        } else {
          if (this.initialAppList.indexOf(appname) === -1) {
            this.activeAppList.push(appname);
          }
          this.openedAppOrder.push(appname);
        }
      }
    },
    getAppState(appname) {
      return (
        this.openedAppOrder.indexOf(appname) !== -1 ||
        this.minApps.indexOf(appname) !== -1
      );
    },
  },
  watch: {
    openedAppOrder: {
      deep: true,
      handler() {
        for (let item of document.getElementsByClassName('activeApp')) {
          item.classList.remove('activeApp');
        }
        let idtext =
          this.openedAppOrder[this.openedAppOrder.length - 1] + 'Task';
        this.$nextTick(() => {
          let dom = document.getElementById(idtext);
          if (dom) dom.classList.add('activeApp');
        });
      },
    },
  },
  created() {
    this.updateTime();
    this.timer = setInterval(this.updateTime, 1000);
  },
  beforeDestroy() {
    clearInterval(this.timer);
  },
};
</script>

<style lang="scss" scoped>
.taskBar {
  z-index: 999;
  height: 39px;
  width: 100%;
  background-color: rgba(243, 243, 243, 0.83);
  backdrop-filter: blur(16px);
  display: flex;
  align-self: flex-end;
  .taskcont {
    display: flex;
    justify-content: center;
    position: relative;
    width: 100%;
    height: 100%;
    .tasksCont {
      display: flex;
      justify-content: center;
      align-items: center;
      .taskIcon {
        display: grid;
        position: relative;
        place-items: center;
        border-radius: 4px;
        margin: 0 2px;
        padding: 0 1px;
        width: 33px;
        height: 33px;
        background-color: rgba(245, 249, 253, 0);
        transition: all 0.3s ease-in-out;
        img {
          width: 22px;
          transition: all 0.2s ease-in-out;
        }
        img.smaller {
          width: 16px;
        }
      }
      .taskIcon.activeApp::after {
        width: 14px;
        background-color: rgb(0, 103, 192);
      }
      .taskIcon::after {
        content: '';
        position: absolute;
        display: block;
        bottom: 0;
        width: 0;
        height: 3px;
        border-radius: 4px;
        background: #858585;
        transition: all 0.2s ease-in-out;
      }
      .taskIcon:hover {
        background-color: rgba(245, 249, 253, 1);
      }
    }
    .taskRight {
      position: absolute;
      top: 0;
      right: 0;
      width: auto;
      height: 100%;
      margin-left: 10px;
      display: flex;
      .taskiconul {
        &:hover {
          background-color: rgb(209, 213, 219);
        }
        padding: 10px 3px 10px 3px;
        cursor: pointer;
        display: flex;
        font-size: 14px;
        .taskiconli {
          width: 25px;
          display: flex;
          justify-content: center;
          align-items: center;
          img {
            width: 17px;
            height: 16px;
          }
        }
      }
      .taskDate {
        height: 100%;
        width: 60px;
        display: flex;
        padding: 0px 4px 4px 4px;
        font-size: 13px;
        flex-direction: column;
        margin-right: 10px;
        &:hover {
          background-color: rgb(209, 213, 219);
        }
        .dateDiv {
          cursor: pointer;
          width: 100%;
          height: 100%;
          padding: 3px 3px 3px 3px;
          text-align: center;
          font-weight: 400;
          margin-bottom: 10px;
        }
      }
    }
  }
}
img {
  -webkit-user-drag: none;
}
</style>
