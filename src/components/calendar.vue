<template>
  <div>
    <a href="#" @click.prevent="showCalendar">
      <slot></slot>
    </a>
    <!-- Modal -->
    <div class="modal fade" id="calendarModal" tabindex="-1" aria-hidden="true">
      <div class="modal-dialog modal-dialog-centered">
        <div class="modal-content rounded-lg overflow-hidden shadow">
          <div class="modal-body p-0">
            <div class="card">
              <div class="card-header bg-primary">
                <nav>
                  <ul class="pagination mb-0 justify-content-center">
                    <li class="page-item">
                      <a class="page-link bg-primary text-white border-0 px-3 py-1" href="#" aria-label="Previous"
                      @click="edditCalendar(-1)">
                        <span aria-hidden="true">&laquo;</span>
                      </a>
                    </li>
                    <li class="page-item w-75 text-center">
                      <span class="d-block bg-white text-primary p-1" style="line-height: 1.25;">{{ calendarTitle }}</span>
                    </li>
                    <li class="page-item">
                      <a class="page-link bg-primary text-white border-0 px-3 py-1" href="#" aria-label="Next"
                      @click="edditCalendar(1)">
                        <span aria-hidden="true">&raquo;</span>
                      </a>
                    </li>
                  </ul>
                </nav>
              </div>

              <table class="table table-hover table-light text-center" @click="selectDate">
                <thead>
                  <tr class="bg-info">
                    <th scope="col">Sun</th>
                    <th scope="col">Mon</th>
                    <th scope="col">Tue</th>
                    <th scope="col">Wed</th>
                    <th scope="col">Thu</th>
                    <th scope="col">Fri</th>
                    <th scope="col">Sat</th>
                  </tr>
                </thead>
                <tbody>
                  <template v-for=" (week, index) in calendarArr">
                    <tr :key="index">
                      <td :class="{'text-primary': typeof(day) == 'string', 'bg-primary': pickDateData.date === day && pickDateData.monthNum === dateData.monthNum}"
                      v-for="(day, key) in week" :key="key"  :data-value="day" :data-type="typeof(day)">{{day}}</td>
                    </tr>
                  </template>
                </tbody>
              </table>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import $ from 'jquery'

export default {
  props: ['passDateData'],
  data () {
    return {
      dateData: {
        year: '',
        month: '',
        monthNum: '',
        date: ''
      },
      calendarArr: [],
      fullDayArr: [],
      allMonthArr: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'],
      pickDateData: Object.assign({}, this.passDateData) // 避免物件傳參考特性，干擾 todoItem 元件編輯功能
    }
  },
  methods: {
    getDate (paraYear, paraMonth) {
      const date = new Date()
      // 元件初次載入透過參數取得日期資料、edditCalendar() 編輯後，則是透過 dateData 物件取得資料
      this.dateData.year = paraYear || this.dateData.year
      this.dateData.monthNum = paraMonth || this.dateData.monthNum
      this.dateData.month = this.allMonthArr[this.dateData.monthNum - 1]
      if (date.getDate().toString().length === 1) {
        this.dateData.date = `0${date.getDate()}`
      } else {
        this.dateData.date = date.getDate().toString()
      }
    },
    getCalendar (day = 1) {
      // 取得目前年份及月份
      const year = this.dateData.year
      const month = this.dateData.monthNum

      // 判斷是否為閏年(2月份天數)：true = 29 天、false = 28 天
      let leapMonth = 28
      if ((year % 4 === 0 && year % 100 !== 0) || (year % 400 === 0)) {
        leapMonth = 29
      }
      const mDay = [31, leapMonth, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31]

      // 蔡勒公式取得星期x
      const c = Math.floor(year / 100)
      let y = year % 100
      let m = month
      const d = day
      if (month === 1 || month === 2) {
        m = month + 12
        y -= 1
      }
      const w = (y + Math.floor(y / 4) + Math.floor(c / 4) - 2 * c + Math.floor((26 * (m + 1)) / 10) + d - 1) % 7

      // 判斷該月份需要 5週35格 OR 6週42格
      let totalLattices = 42
      let totalWeeks = 6
      if (w + mDay[month - 1] <= 35) {
        totalLattices = 35
        totalWeeks = 5
      }
      // 計算月曆 35/42格中 本月 "前及後" 的所佔天數
      const beforeMonthDay = w
      const afterMonthDay = totalLattices - mDay[month - 1] - w

      // 進行 Array.Push() 前先確保內容清空( 避免與 edditCalendar() 產生衝突 )
      this.fullDayArr = []
      this.calendarArr = []
      // 非本月日期刻意使用字串型別的變數傳入陣列做為區隔 並且為CSS樣式的判別條件
      if (beforeMonthDay) {
        let originateDay = mDay[month - 2] - beforeMonthDay + 1
        let endDay = mDay[month - 2] + 1
        if (month === 1) {
          originateDay = mDay[11] - beforeMonthDay + 1
          endDay = mDay[11] + 1
        }
        for (let i = originateDay; i < endDay; i++) {
          this.fullDayArr.push(i.toString())
        }
      }

      for (let i = 1; i < mDay[month - 1] + 1; i++) {
        this.fullDayArr.push(i)
      }

      if (afterMonthDay) {
        for (let i = 1; i < afterMonthDay + 1; i++) {
          this.fullDayArr.push(i.toString())
        }
      }
      // 將 fullDayArr 分割成 5 or 6週之陣列
      for (let i = 0; i < totalWeeks; i++) {
        this.calendarArr.push(this.fullDayArr.slice(i * 7, (i + 1) * 7))
      }
    },
    edditCalendar (num) {
      // 月份 及 跨年份 資料編輯
      this.dateData.monthNum += num
      if (this.dateData.monthNum === 0) {
        this.dateData.year -= 1
        this.dateData.monthNum = 12
      } else if (this.dateData.monthNum === 13) {
        this.dateData.year += 1
        this.dateData.monthNum = 1
      }
      // 重新渲染月曆介面
      this.getDate()
      this.getCalendar()
    },
    selectDate (event) {
      if (event.target.nodeName === 'TD') {
        const type = event.target.dataset.type
        const value = event.target.dataset.value
        const vm = this
        // 當欲釘選的日期數值型別為字串時，表示非當月日期 則月曆前往至該月份
        if (type === 'string' && Number(value) > 15) {
          vm.edditCalendar(-1)
        } else if (type === 'string' && Number(value) < 15) {
          vm.edditCalendar(1)
        } else {
          // 釘選當月日期 將對應數值寫入
          vm.fullDayArr.forEach(function (item, index) {
            if (item === Number(value)) {
              vm.pickDateData.date = item
              vm.pickDateData.monthNum = vm.dateData.monthNum
              vm.pickDateData.year = vm.dateData.year
            }
          })
          vm.$emit('passPickDate', vm.pickDateData)
          $('#calendarModal').modal('hide')
        }
      }
    },
    showCalendar () {
      this.getDate(this.pickDateData.year, this.pickDateData.monthNum)
      this.getCalendar()
      $('#calendarModal').modal('show')
    }
  },
  computed: {
    calendarTitle () {
      return `${this.dateData.month} - ${this.dateData.year}`
    }
  }
}
</script>

<style leng="scss">

  td:hover {
    background: #d2c8c3;
    cursor: pointer;
  }
</style>
