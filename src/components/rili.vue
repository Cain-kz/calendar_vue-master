
<template>
  <div class="rili">
    <!-- 工具栏 star -->
    <div class="tools">
      <div class="toolsLeft flex">
        <div class="jiantou" @click="arrowLeft">
           <i class="left"></i>
        </div>
        <div class="jiantouCenter" >
          <div @click="clickMonth"> {{TIME | defaultTimeFormat }}</div>
        </div>
         <div class="jiantou" @click="arrow">
           <i class="right"></i>
        </div>
      </div>
      <div class="toolsRight">
        <div class="btn flex-center" @click="Back">今天</div>
      </div>
    </div>
    <!-- 工具栏 end -->

    <!-- 日历头部 star  -->
    <div class="title">
      <div v-for="(item, index) in weekTitle" :key="index" class="">
        {{ item }}
      </div>
    </div>
    <!-- 日历头部 end  -->

    <div class="body" >
      <!-- 上个月剩余日期 star -->
      <div
        class="box disabled"
        v-for="(item, index) in topMonthList"
        :key="'top'+index"

        @click="arrowLeft"
      >

        <div class="gongli">
            {{ item.day }}
        </div>
        <div class="nongli">
          <span v-if="item.yangEvents.yangeventName" class="bg-blue fw">{{item.yangEvents.yangeventName}}</span>
            <span v-else >{{item.calendar.IDayCn}}</span>
        </div>
      </div>
      <!-- 上个月剩余日期 end -->

      <!-- 当前月 日期 star -->
      <div
        class="box"
        :class="{'selected':selecID == index && getTimePosition() || selecID2 == index,'custom':item.custom, 'Multiple':item.Multiplem,'disabled2':item.historicalBol}"
        v-for="(item, index) in this_DateList"
        :key="index"
        @click="selected(item,index)"
      >
       <div class="gongli">
            {{ item.day }}
        </div>
        <div class="nongli">
            <span v-if="item.yangEvents.yangeventName" class="bg-blue fw">{{item.yangEvents.yangeventName}}</span>
            <span v-else >{{item.calendar.IDayCn}}</span>
        </div>

      </div>
      <!-- 当前月 日期 end -->

      <!-- 下个月剩余日期 star -->
      <div
        class="box disabled"
        v-for="(item, index) in bottomMonthList"
        :key="'bottom'+index"

        @click="arrow"
      >
        <div class="gongli" :class="{'today fw': item.today}">
            {{ item.day }}
        </div>
        <div class="nongli">
            <span v-if="item.yangEvents.yangeventName" class="bg-blue fw">{{item.yangEvents.yangeventName}}</span>
            <span v-else >{{item.calendar.IDayCn}}</span>
        </div>
      </div>
      <!-- 下个月剩余日期 end -->
    </div>

    <!-- 月份弹出层 -->
    <transition name="van-fade">
         <div class="Month fs-xxxxl " v-show="MonthBoxBol" @click="MonthBoxBol=false">
           <div class="w-100 h-10 jc-center flex">
              {{TIME | MonthFormat }}
           </div>
           <div class="flex flex-w jc-around w-100 h-90">
             <div class="w-30 h-25 box1 flex-center fs-xxxxl" v-for="(item,index) in 12 " :key="index" @click="clickMonthBox(index+1)">{{index+1}} 月</div>
           </div>
        </div>
    </transition>

    <div class="line">
      <div class="flex-time">
        <div>开始时间：</div>
        <div class="select-time">
          <select v-model="selectedHour">
            <option v-for="hour in hours" :key="hour" :value="hour">
              {{ padZero(hour) }}
            </option>
          </select>
          :
          <select v-model="selectedMinute">
            <option v-for="minute in minutes" :key="minute" :value="minute">
              {{ padZero(minute) }}
            </option>
          </select>
        </div>
      </div>
      <div class="flex-time">
        <div>结束时间：</div>
        <div class="select-time">
          <select v-model="selectedEndHour">
            <option v-for="hour in hours" :key="hour" :value="hour">
              {{ padZero(hour) }}
            </option>
          </select>
          :
          <select v-model="selectedEndMinute">
            <option v-for="minute in minutes" :key="minute" :value="minute">
              {{ padZero(minute) }}
            </option>
          </select>
        </div>
      </div>
    </div>
  </div>
</template>

<script>

import calendar from '@/utils/calendar.js'
export default {
  props: {
    // 默认日期
    defaultTime: {
      type: String,
      default: function () {
        return `${new Date().getFullYear()}-${new Date().getMonth() + 1}-${new Date().getDate()}`
      }
    },
    // 自定义日期
    customList: {
      type: Array,
      default: function () {
        return []
      }
    },
    // 是否多选
    Multiple: {
      type: Boolean,
      default: false
    },
    // 超过当前时间不可选择 默认true可以选择
    historicalDate: {
      type: Boolean,
      default: true
    }
  },
  filters: {
    // 默认日期过滤
    defaultTimeFormat: function (value) {
      var defaultTime = value.split('-')
      defaultTime[0] = parseInt(defaultTime[0])
      defaultTime[1] = parseInt(defaultTime[1])
      return `${defaultTime[0]}年  ${defaultTime[1]}月`
    },
    // 月选择 日期过滤
    MonthFormat: function (value) {
      var defaultTime = value.split('-')
      defaultTime[0] = parseInt(defaultTime[0])
      return `${defaultTime[0]}年`
    }
  },
  watch: {
    TIMELIST: function (newVal) {
      newVal[0] = parseInt(newVal[0])
      newVal[1] = parseInt(newVal[1])
      newVal[2] = parseInt(newVal[2])
    },
    TIME: function () {
      this.init()
    },
    customList: function (val) {
      this.CUSTOMLIST = val
      this.init()
    }
  },

  data () {
    return {
      TIME: '', // 获取props的defaultTime
      TIMELIST: [], // 分割日期
      CUSTOMLIST: [], // 获取props的customList

      weekTitle: ['S', 'M', 'T', 'W', 'T', 'F', 'S'],

      MonthBoxBol: false, // 月份弹出层

      this_Year: new Date().getFullYear(), // 当前年
      this_Month: new Date().getMonth() + 1, // 当前月
      this_Day: new Date().getDate(), // 当前日

      // 日期集合
      this_DateList: [], // 当前月日期集合
      topMonthList: [], // 上个月剩余日期集合
      bottomMonthList: [], // 下个月剩余日期集合

      selecID: null, // 当前选中index
      selecID2: null, // 当多选时启用
      selectedVal: null, // 当前选中的信息

      MultipleList: [], // 多选时存放的 开始日期和结束日期

      yangevents: { // 阳历节日
        '0101': ' 元旦',
        '0214': ' 情人节',
        '0308': ' 妇女节',
        '0312': ' 植树节',
        '0315': ' 消费者权益日',
        '0401': ' 愚人节',
        '0501': ' 劳动节',
        '0504': ' 青年节',
        '0512': ' 护士节',
        '0601': ' 儿童节',
        '0701': ' 建党节',
        '0801': ' 建军节',
        '0910': ' 教师节',
        '0928': ' 孔子诞辰',
        1001: ' 国庆节',
        1006: ' 老人节',
        1024: ' 联合国日',
        1224: ' 平安夜',
        1225: ' 圣诞节'
      },

      selectedHour: '1',
      selectedMinute: '23',
      selectedEndHour: '',
      selectedEndMinute: ''
    }
  },

  computed: {
    // 当前月
    currentMonth () {
      return this.this_Month
    },
    // 当前年
    currentYear () {
      return this.this_Year
    }
  },

  components: {},

  computed: {
    hours: () => {
      return Array.from({ length: 24 }, (_, i) => i)
    },
    minutes: () => {
      return Array.from({ length: 60 }, (_, i) => i)
    }
  },

  created () {
    this.TIME = this.defaultTime
    this.CUSTOMLIST = this.customList
    this.init()
    console.log(this.historicalDate)
  },

  mounted () {},

  methods: {
    padZero (num) {
      return num < 10 ? '0' + num : num
    },
    // 返回今天
    Back () {
      this.TIME = `${this.this_Year}-${this.this_Month}-${this.this_Day}`
      this.$emit('result', {
        year: this.zeroPad(this.this_Year),
        month: this.zeroPad(this.this_Month),
        day: this.zeroPad(this.this_Day)
      })
      this.init()
    },
    // 日期初始化
    init () {
      this.this_DateList = []
      this.topMonthList = []
      this.bottomMonthList = []

      var time = this.TIMELIST = this.TIME.split('-')
      time[1] = this.zeroPad(time[1])
      // 获取上个月剩余日期集合
      var firstDate = new Date(time[0], time[1] - 1).getDay() // 当前月第一天是星期几
      var topMonthLength = new Date(
        time[0],
        time[1] - 1,
        0
      ).getDate() // 上个月月数
      var thisTopMonth = topMonthLength - firstDate // 求上个月剩余几天
      for (let i = 1; i <= firstDate; i++) {
        this.topMonthList[i - 1] = {
          year: time[1] === 1 ? time[0] - 1 : time[0],
          month: this.zeroPad(time[1] === 1 ? 12 : parseInt(time[1]) - 1), // 如果本月是一月，那么上个月就是去年的12月
          day: this.zeroPad(thisTopMonth + i)
        }

        this.topMonthList[i - 1].calendar = calendar.solar2lunar(this.topMonthList[i - 1].year, this.topMonthList[i - 1].month, this.topMonthList[i - 1].day) // 公历转农历
        this.topMonthList[i - 1].yangEvents = this.getyangEvents('', this.topMonthList[i - 1].month, this.topMonthList[i - 1].day) // 公历转农历
        // 是否可选日历日期

        // this.topMonthList[i - 1].historicalBol = this.historicalDate_M('', this.topMonthList[i - 1].month, this.topMonthList[i - 1].day)
      }

      // 获取当前月
      var thisDateLength = new Date(
        time[0],
        time[1],
        0
      ).getDate()
      for (let i = 1; i <= thisDateLength; i++) {
        this.this_DateList[i - 1] = {
          year: time[0],
          month: time[1],
          day: this.zeroPad(i),
          calendar: calendar.solar2lunar(time[0], time[1], i)
        }
        // if (this.this_Day == i) { // 今天默认选中
        //   this.selecID = i - 1
        // }
        this.this_DateList[i - 1].yangEvents = this.getyangEvents('', this.this_DateList[i - 1].month, this.this_DateList[i - 1].day) // 公历转农历
        this.this_DateList[i - 1].custom = this.getCustomList(this.this_DateList[i - 1].year, this.this_DateList[i - 1].month, this.this_DateList[i - 1].day)// 自定义日期

        // 标记今天
        if (
          this.this_Day === i &&
                this.this_Year === time[0] &&
                this.this_Month === time[1]
        ) {
          this.this_DateList[i - 1].today = true
        }

        if (this.historicalDate) {
          continue
        }
        this.this_DateList[i - 1].historicalBol = this.historicalDate_M(this.this_DateList[i - 1].year, this.this_DateList[i - 1].month, this.this_DateList[i - 1].day)// 自定义日期
      }
      // 获取下个月剩余日期集合
      var thisBootomMonth =
        42 - this.topMonthList.length - this.this_DateList.length // 求下个月剩余几天
      for (let i = 1; i <= thisBootomMonth; i++) {
        this.bottomMonthList[i - 1] = {
          year: time[1] === 12 ? time[0] + 1 : time[0], // 如果本月是十二月，那么下个月就是明年的1月
          month: this.zeroPad(time[1] === 12 ? 1 : parseInt(time[1]) + 1), // 如果本月是十二月，那么下个月就是明年的1月
          day: this.zeroPad(i)
        }

        this.bottomMonthList[i - 1].calendar = calendar.solar2lunar(this.bottomMonthList[i - 1].year, this.bottomMonthList[i - 1].month, this.bottomMonthList[i - 1].day) // 公历转农历
        this.bottomMonthList[i - 1].yangEvents = this.getyangEvents('', this.bottomMonthList[i - 1].month, this.bottomMonthList[i - 1].day) //
      }
      this.$forceUpdate()
    },

    // 获取前后年份 生成集合
    yearInit () {
    },
    // 判断是否历史日期是否可选
    historicalDate_M  (y, m, d) {
      var val = `${y}-${m}-${d}`
      // 当前日期
      var current = `${this.this_Year}-${this.this_Month}-${this.this_Day}`
      var oDate1 = new Date(val)
      var oDate2 = new Date(current)
      if (oDate1.getTime() < oDate2.getTime()) {
        return true
      }
      return false
    },

    // 选中日期事件
    selected (item, index) {
      // 以往日期不可选
      if (item.historicalBol) {
        return
      }
      var data = []
      var length = this.this_DateList.length

      if (this.Multiple) {
        if ((this.selecID && this.selecID2) || (this.selecID === 0 && this.selecID2)) {
          this.selecID = this.selecID2 = null
          this.MultipleList = []
          for (let i = 0; i < length; i++) {
            this.this_DateList[i].Multiple = false
          }
        } else if (this.selecID == null) {
          this.selecID = index
          this.MultipleList[0] = item
        } else if (this.selecID || this.selecID === 0) {
          this.selecID2 = index
          this.MultipleList[1] = item
          data = this.getDiffDate(
            `${this.MultipleList[0].year}-${this.MultipleList[0].month}-${this.MultipleList[0].day}`,
            `${this.MultipleList[1].year}-${this.MultipleList[1].month}-${this.MultipleList[1].day}`
          )

          if (data.length > 0) {
            data.push(`${this.MultipleList[1].year}-${this.MultipleList[1].month}-${this.MultipleList[1].day}`)
          }

          for (let i = 0; i < data.length; i++) {
            for (let j = 0; j < length; j++) {
              if (
                  `${this.this_DateList[j].year}-${this.this_DateList[j].month}-${this.this_DateList[j].day}` ===
                  data[i]
              ) {
                this.this_DateList[j].Multiple = true
                continue
              }
            }
          }

          this.$emit('result', data)
        }
      } else {
        this.selecID = index
        this.selectedVal = item
        this.$emit('result', item)
      }
    },
    // 格式化日期
    getDate (datestr) {
      var temp = datestr.split('-')
      if (temp[1] === '01') {
        temp[0] = parseInt(temp[0], 10) - 1
        temp[1] = '12'
      } else {
        temp[1] = parseInt(temp[1], 10) - 1
      }
      // new Date()的月份入参实际都是当前值-1
      var date = new Date(temp[0], temp[1], temp[2])
      return date
    },
    // 求两日期之间的时间
    getDiffDate (start, end) {
      var startTime = this.getDate(start)
      var endTime = this.getDate(end)
      var dateArr = []
      while ((endTime.getTime() - startTime.getTime()) > 0) {
        var year = startTime.getFullYear()
        var month = (startTime.getMonth() + 1).toString().length === 1 ? '0' + (parseInt(startTime.getMonth().toString(), 10) + 1) : (startTime.getMonth() + 1)
        var day = startTime.getDate().toString().length === 1 ? '0' + startTime.getDate() : startTime.getDate()
        dateArr.push(year + '-' + month + '-' + day)
        startTime.setDate(startTime.getDate() + 1)
      }
      return dateArr
    },
    // 保存当前日期位置

    getTimePosition () {
      var time = this.TIME.split('-')
      return `${this.selectedVal.year}-${this.zeroPad(this.selectedVal.month)}` === `${time[0]}-${this.zeroPad(time[1])}`
    },

    // 获取自定义阳历事件
    getyangEvents (y, m, d) {
      if (Object.keys(this.yangevents).length === 0) { return false }
      const eventName = this.yangevents[this.zeroPad(m) + '' + this.zeroPad(d)]
      const data = {}
      if (eventName !== undefined) {
        data.yangeventName = eventName
      }
      return data
    },
    // 获取自定义日期
    getCustomList (y, m, d) {
      var val = `${y}-${m}-${d}`

      if (Object.keys(this.CUSTOMLIST).length === 0) return false
      for (let i = 0; i < this.CUSTOMLIST.length; i++) {
        if (val === this.CUSTOMLIST[i]) {
          return this.CUSTOMLIST[i]
        }
      }

      return false
    },
    // 日期补零
    zeroPad (n) {
      // eslint-disable-next-line no-unused-expressions
      typeof n !== 'string' ? n = String(n) : n
      return n.padStart(2, '0')
    },

    // 上一页
    arrowLeft () {
      if (this.TIMELIST[1] === 1) {
        this.TIME = `${this.TIMELIST[0] - 1}-12-${this.TIMELIST[2]}`
      } else {
        this.TIME = `${this.TIMELIST[0]}-${this.TIMELIST[1] - 1}-${this.TIMELIST[2]}`
      }
      this.TIMELIST = this.TIME.split('-')
    },
    // 下一页
    arrow () {
      if (this.TIMELIST[1] === 12) {
        this.TIME = `${this.TIMELIST[0] + 1}-1-${this.TIMELIST[2]}`
      } else {
        this.TIME = `${this.TIMELIST[0]}-${this.TIMELIST[1] + 1}-${this.TIMELIST[2]}`
      }
      this.TIMELIST = this.TIME.split('-')
    },

    // 月份弹出层事件
    clickMonth () {
      this.MonthBoxBol = true
    },
    // 选中月份
    clickMonthBox (val) {
      this.TIME = `${this.this_Year}-${val}-01`
    },
    setSTime (type) {
      if (type === 'start') {

      } else {

      }
    }
  }
}
</script>
<style lang='scss' scoped>
i {
  border: solid black;
  border-width: 0 1px 1px 0;
  display: inline-block;
  padding: 3px;
}
.right {
  transform: rotate(-45deg);
  -webkit-transform: rotate(-45deg);
}
.left {
  transform: rotate(135deg);
  -webkit-transform: rotate(135deg);
}
.rili {
  width: 100%;
  display: flex;
  flex-direction: column;
  position: relative;
}
.rili .title {
  width: 100%;
  height: 1rem;
  display: flex;
  font-size: 14px;
}
.rili .title div {
  width: 13.7%;
  border: 1px solid #fff;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}
.flex{
  display: flex;
}
.body {
  width: 100%;
  font-size: 16px;
  display: flex;
  flex-wrap: wrap;
}
.body .box {
  width: 13%;
  height:3rem;
  display: flex;
  justify-content: center;
  align-items: center;
  margin: 0.3%;
  flex-direction: column;
  position: relative;
  border-radius: 3px;
  overflow: hidden;
  border: 1px solid #fff;
}
// 禁用
.disabled {
  color: rgb(218, 218, 218) !important;
}
// 当前月份禁用
.disabled2 {
  color: rgb(160, 160, 160) !important;
}
.tools {
  width: 100%;
  display: flex;
  font-size: 12px;
}
.toolsLeft {
  width: 80%;
}
.toolsRight {
  width: 20%;
}
.toolsRight .btn {
  padding: 10px;
  box-sizing: border-box;
  background: #f9f9f9;
  color: #999999;
}
.today {
  //border: 1px solid #f89595;
  background: #3783D6;
  //color: rgb(255, 104, 104);
}
.selected {
  background: #FC9D9B !important;
}
.nongli{
    font-size: 12px;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
}
.bg-blue{
  color: rgb(255, 157, 157);
}
.fw{
  font-weight: bold;
}

.jiantou{
  width: 10%;
  font-size: 14px;
  display: flex;
  justify-content: center;
  align-items: center;
}
.jiantouCenter{
  width: 80%;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 16px;
}

//默认
.spot_border{
  border: 1px solid #999999;
}

.Month{
    position: absolute;
    width: 100%;
    height: 100%;
    background: #fff;
    transition: all 0.5s;
    z-index: 2;
    font-size: 16px;
}
.year{
    position: absolute;
    width: 100%;
    height: 100%;
    background: #fff;
    transition: all 0.5s;
    z-index: 3;
    overflow: auto;
}
.box1{
  box-shadow:2px 2px 10px #d3d2d2;
}

.gongli,.nongli{

  height: 30%;
}
//自定义颜色
.custom{
  border: 1px solid #9bcbff !important;
}
//多选颜色
.Multiple{
  background: #ff6b57;
}

.line{
  border-top: 1px solid #d3d2d2;
}
.flex-time{
  display: flex;
}
</style>
