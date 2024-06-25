<template>
  <div class="home">
    <calendar @result="result" :customList="customList" v-if="customBol"/>
  </div>
</template>

<script>
// @ is an alias to /src
import calendar from '@/components/rili.vue'

export default {
  name: 'Home',
  components: {
    calendar
  },
  data () {
    return {
      customBol: false,
      customList: []
    }
  },
  created () {
    this.customBol = true
    this.customList = [this.formatDate(new Date())]
  },
  methods: {
    formatDate (date) {
      const year = date.getFullYear()
      let month = (date.getMonth() + 1).toString() // 月份从0开始，所以需要加1
      let day = date.getDate().toString()

      if (month.length < 2) {
        month = '0' + month
      }
      if (day.length < 2) {
        day = '0' + day
      }

      return `${year}-${month}-${day}`
    },
    result (val) {
      console.log(val)
    },
    gtouchstart (e) {
      console.log(e)
      console.log('1，按下啦啦啦啦啦')
    },
    gtouchmove (e) {
      console.log(e)
      console.log('2，按下并且在移动呢')
    },
    gtouchend () {
      console.log('3，松开啦啦啦啦啦')
    }
  }
}
</script>
