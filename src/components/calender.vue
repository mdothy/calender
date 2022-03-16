<template>
  <div>
    <div class="calender-wrapper">
      <div class="box">
        <div class="box-left">
          <div class="box-count-item" v-for="(item,index) in time" :key="index">{{item}}</div>
        </div>
        <div class="box-right">
          <div class="row-container">
            <div class="row-item" v-for="(item,index) in count" :key="index"></div>
          </div>

          <div class="current-time" :style="`top:${currentLineTop}px`">
            <div class="current-time-left">{{current}}</div>
            <div class="current-time-direction"></div>
            <div class="current-time-line"></div>
          </div>

          <div class="events-container" @click="handleCreateItem($event)">
            <div
              class="event-instance-past-wrapper"
              :style="increase ? 'position:relative;' : 'display:none' "
            >
              <div
                class="item-show"
                :style="increase ? `top:${offsetY}px;height:${itemHeight}px` : ''"
              >
                <div class="summary property" style="height:24px"></div>
              </div>
            </div>

            <div style="position:relative">
              <div
                v-for="(item,index) in itemData"
                :key="index"
                draggable="true"
                @dragstart="onDragStart($event,item,index)"
                @dragend="onDragEnd($event,item)"
                class="child-item"
                :style="`top:${item.offsetY}px;height:${item.height}px`"
                @click="handleItemClick($event,item)"
              >
                <div @click="handleShowItem($event,item)">
                  <span style="font-size:14px;color:rgb(28,76,186">{{item.content}}</span>
                  <!-- <span style="font-size:14px">{{item.content}}</span> -->
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <el-dialog title="提示" :visible.sync="visible" width="30%" :before-close="handleClose">
      <el-form :label-position="labelPosition" label-width="40px" :model="formLabelAlign">
        <el-form-item label="名称">
          <el-input size="small" v-model="formLabelAlign.name"></el-input>
        </el-form-item>

        <el-row span="24">
          <el-form-item label="时间">
            <el-time-select
              style="width:47%;margin-right:4%"
              v-model="formLabelAlign.timeStart"
              size="small"
              @change="handlePreTime($event)"
              :picker-options="{start: '00:00',step: '00:30', end: '23:30'}"
              placeholder="请选择时间"
            ></el-time-select>
            <el-time-select
              style="width:47%"
              v-model="formLabelAlign.timeEnd"
              size="small"
              @change="handleNextTime($event)"
              :picker-options="{start: instanceStartTime,step: '00:30', end: '23:30'}"
              placeholder="请选择时间"
            ></el-time-select>
          </el-form-item>
        </el-row>
        <!-- <el-form-item label="活动区域">
          <el-input v-model="formLabelAlign.region"></el-input>
        </el-form-item>
        <el-form-item label="活动形式">
          <el-input v-model="formLabelAlign.type"></el-input>
        </el-form-item>-->
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="visible = false">取 消</el-button>
        <el-button type="primary" @click="handleSubmit">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="提示" :visible.sync="itemvisible" width="30%" :before-close="handleItemClose">
      <el-form :label-position="labelPosition" label-width="80px" :model="formLabelAlign">
        <el-form-item label="名称">
          <el-input :disabled="true" v-model="formLabelAlign.name"></el-input>
        </el-form-item>
        <!-- <el-form-item label="活动区域">
          <el-input v-model="formLabelAlign.region"></el-input>
        </el-form-item>
        <el-form-item label="活动形式">
          <el-input v-model="formLabelAlign.type"></el-input>
        </el-form-item>-->
      </el-form>
    </el-dialog>
  </div>
</template>

<script>
export default {
  name: "calender",
  data() {
    return {
      time: [
        "01:00",
        "02:00",
        "03:00",
        "04:00",
        "05:00",
        "06:00",
        "07:00",
        "08:00",
        "09:00",
        "10:00",
        "11:00",
        "12:00",
        "13:00",
        "14:00",
        "15:00",
        "16:00",
        "17:00",
        "18:00",
        "19:00",
        "20:00",
        "21:00",
        "22:00",
        "23:00"
      ],
      moment: [
        "00:00",
        "00:30",
        "01:00",
        "01:30",
        "02:00",
        "02:30",
        "03:00",
        "03:30",
        "04:00",
        "04:30",
        "05:00",
        "05:30",
        "06:00",
        "06:30",
        "07:00",
        "07:30",
        "08:00",
        "08:30",
        "09:00",
        "09:30",
        "10:00",
        "10:30",
        "11:00",
        "11:30"
      ],
      instanceStartTime: "00:00",
      timer: "",
      current: "",
      currentLineTop: 0,
      itemvisible: false,
      itemData: [],
      increase: false,
      offsetY: 0,
      itemHeight: 24,
      startIndex: 0,
      endIndex: 0,
      count: 24,
      dragItem: "",
      dragIndex: 0,
      visible: false,
      labelPosition: "right",
      formLabelAlign: {
        name: "主题",
        region: "",
        type: "",
        timeStart: "",
        timeEnd: ""
      }
    };
  },
  props: {
    msg: String
  },
  mounted() {
    this.initMoment();
    this.timer = setInterval(() => this.getCurrentTime(), 1000);
  },
  methods: {
    initMoment() {
      // for(let i = 0 ; i < 24 ;i++){
      //   let pre = Math.floor(i / 2)
      //   let preValue = pre >= 10 ?  (i )  : '0' + pre
      // //   // pre = i % 2 == 0 ? pre + ':00' : pre + ':30'
      // //   // let next = i % 2
      //   this.moment[i] = i % 2 == 0 ? preValue : ''
      //   console.log(this.moment)
      // }
    },
    getCurrentTime() {
      let date = new Date();
      let hour = date.getHours() > 10 ? date.getHours() : "0" + date.getHours();
      let minute =
        date.getMinutes() > 10 ? date.getMinutes() : "0" + date.getMinutes();
      this.current = hour + ":" + minute;
      this.currentLineTop = Number(hour) * 49 + (Number(minute) / 60) * 49;
      return true;
    },
    handleCreateItem(event) {
      this.increase = true;
      this.visible = true;
      this.offsetY = Math.floor(event.offsetY / 24.5) * 24.5;
      this.startIndex = Math.floor(event.offsetY / 24.5)
      this.endIndex = this.startIndex + 1
      let currentDirection = Math.floor(event.offsetY / 24.5);
      this.formLabelAlign.timeStart = this.moment[currentDirection];
      let instanceTime = this.moment.slice(currentDirection);
      this.formLabelAlign.timeEnd = instanceTime[1];
      this.instanceStartTime = instanceTime[0];
    },
    handlePreTime(time) {
      let index = 0;
      for (let i = 0; i < this.moment.length; i++) {
        if (this.moment[i] == time) {
          index = i;
          break;
        }
      }
      let instanceTime = this.moment.slice(index);
      this.instanceStartTime = instanceTime[1];
      this.formLabelAlign.timeEnd = instanceTime[1];
      this.startIndex = index;
      this.offsetY = 24.5 * index;
    },
    handleNextTime() {
      let index = 0;
      for (let i = 0; i < this.moment.length; i++) {
        if (this.moment[i] == this.formLabelAlign.timeEnd) {
          index = i;
          break;
        }
      }
      this.itemHeight = (index - this.startIndex) * 24.5;
    },
    handleClose() {
      this.visible = false;
    },
    handleItemClose() {
      this.itemvisible = false;
    },
    handleSubmit() {
      this.increase = false;
      this.visible = false;
      this.itemData.push({
        increase: this.increase,
        offsetY: this.offsetY,
        height:this.itemHeight,
        content: this.formLabelAlign.name
      });
    },
    handleShowItem() {
      this.itemvisible = true;
    },
    onDragStart(event, item, index) {
      this.dragItem = item;
      this.dragIndex = index;
      event.dataTransfer.dropEffect = "move";
    },
    onDrag(event) {
      console.log(event, 1);
    },
    onDragEnd(event) {
      if (event.clientY < 20) {
        this.itemData[this.dragIndex].offsetY = 1;
      } else {
        this.itemData[this.dragIndex].offsetY =
          Math.floor(event.y / 24.5) * 24.5;
      }
    },
    handleItemClick(event) {
      event.stopPropagation();
      event.preventDefault();
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.calender-wrapper {
  width: 100%;
  height: 100%;
}
.box {
  width: 80%;
  margin: 0 auto;
  display: flex;
}
.box-left {
  width: 60px;
  padding: 40px 0 0 0;
  display: flex;
  flex-direction: column;
  justify-content: space-between;
}
.box-right {
  position: relative;
  width: calc(100% - 62px);
  border-left: solid 1px #ddd;
  border-right: solid 1px #ddd;
}
.row-container {
  position: absolute;
  left: 0;
  top: 0;
  pointer-events: none;
  width: 100%;
}
.current-time {
  position: absolute;
  width: 100%;
  height: 1px;
}
.current-time-line {
  background: red;
  height: 1px;
  width: 100%;
}
.current-time-direction {
  left: -5px;
  width: 10px;
  position: absolute;
  /* left: -10px; */
  background: red;
  height: 10px;
  border-radius: 50%;
  top: -5px;
}
.current-time-left {
  color: #f54a45;
  position: absolute;
  left: 8px;
  font-size: 14px;
  top: -10px;
}
.row-item {
  border-bottom: solid 1px #ddd;
  text-align: left;
  height: 48px;
}
/* .row-item:last-child {
  border-bottom: none;
} */

.child-item {
  position: absolute;
  width: 100%;
  background-color: rgb(225, 234, 255);
  text-align: left;
  border: solid 1px #a2b2d7;
}
.events-container {
  width: 100%;
  height: 100%;
  border-top: solid 1px #ddd;
  align-items: flex-start;
  justify-content: space-between;
  position: relative;
  overflow: scroll;
  box-sizing: border-box;
  overflow-y: overlay;
  overflow-x: hidden;
}
.item-show {
  position: absolute;
  line-height: 15px;
  background-color: rgb(225, 234, 255);
  color: rgb(36, 91, 219);
  border: 0.5px solid rgb(51, 112, 255);
  opacity: 0.9;
  border-radius: 1px;
  box-shadow: rgba(0, 0, 0, 0.2) 0px 2px 4px 0px;
  width: 100%;
}
.box-count-item {
  height: 49px;
  font-size: 14px;
  text-align: center;
  color: rgba(0, 0, 155, 0.6);
}
</style>
