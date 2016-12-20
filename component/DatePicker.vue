<template>
    <div class="date-picker">
        <div v-if="active_block" class="picker--block">
            <div class="picker--header">
                <div class="header--toggle toggle--left" @click="incrementBlock('down')">
                    <img src="/image/ic_keyboard_arrow_left_black_24px.svg" alt="left">
                </div>
                <div class="header--reading">
                    <div class="reading--date" v-if="type == 'date'">
                        <span class="date date--month">{{active_block.month | filterLanguageMonth(set_language, active_block.month)}}</span>
                        <span v-if="day_display" class="date date--day">{{day_display}}</span>
                        <span class="date date--year">{{active_block.year}}</span>
                    </div>
                    <div class="reading--date-range" v-if="type == 'range'">
                        <div class="range range--start">
                            <template v-if="date_range_select.start.date_display.day">
                                <span class="date date--month">{{date_range_select.start.date_display.month | filterLanguageMonth(set_language, active_block.month)}}</span>
                                <span class="date date--day">{{date_range_select.start.date_display.day}}</span>
                                <span class="date date--year">{{date_range_select.start.date_display.year}}</span>
                            </template>
                            <template v-else>
                                <span class="date date--month">{{active_block.month | filterLanguageMonth(set_language, active_block.month)}}</span>
                                <span class="date date--year">{{active_block.year}}</span>
                            </template>
                        </div>
                        <span v-show="date_range_select.end.date_display.day">-</span>
                        <div class="range range--end" v-show="date_range_select.end.date_display.day">
                            <template v-if="date_range_select.end.date_display.day">
                                <template v-show="date_range_select.end.date_display.day">
                                    <span class="date date--month">{{date_range_select.end.date_display.month | filterLanguageMonth(set_language, active_block.month)}}</span>
                                    <span class="date date--day">{{date_range_select.end.date_display.day}}</span>
                                    <span class="date date--year">{{date_range_select.end.date_display.year}}</span>
                                </template>
                            </template>
                            <template v-else>
                                <span class="date date--month">{{active_block.month | filterLanguageMonth(set_language, active_block.month)}}</span>
                                <span class="date date--year">{{active_block.year}}</span>
                            </template>
                        </div>
                    </div>
                </div>
                <div class="header--toggle toggle--right" @click="incrementBlock('up')">
                    <img src="/image/ic_keyboard_arrow_right_black_24px.svg" alt="right">
                </div>
            </div>
            <div class="picker--grid">
                <div class="grid--week">
                    <span class="day" v-for="day in active_block.days">{{day | filterLanguageWeek(set_language, day)}}</span>
                </div>
                <div class="grid--dates">
                    <div class="group--container" :class="{'container--next': isNext, 'container--prev': isPrev, 'transition-next': isTransition}">
                        <div class="date--group">
                            <span class="date--block grey-date" v-for="date in active_block.prev_block.day_offset">{{date.display}}</span>
                            <span class="date--block" v-for="date in active_block.prev_block.dates" :class="{'active--day': date_select == date.date_time || date_range_select.start.date_time == date.date_time || date_range_select.end.date_time == date.date_time || isStart && date_range_select.start.date_time <= date.date_time && date_range_select.end.date_time >= date.date_time}">{{date.display}}</span>
                            <span class="date--block grey-date" v-for="date in active_block.prev_block.day_fillIn">{{date.display}}</span>
                        </div>
                        <div class="date--group">
                            <span class="date--block grey-date" v-for="date in active_block.day_offset" @click="incrementBlock('down')" :class="{'active--day': date_select == date.date_time || date_range_select.start.date_time == date.date_time || date_range_select.end.date_time == date.date_time || isStart && date_range_select.start.date_time <= date.date_time && date_range_select.end.date_time >= date.date_time}">{{date.display}}</span>
                            <span class="date--block" v-for="date in active_block.dates" @click="selectDate(date)" :class="{'active--day': date_select == date.date_time || date_range_select.start.date_time == date.date_time || date_range_select.end.date_time == date.date_time || isStart && date_range_select.start.date_time <= date.date_time && date_range_select.end.date_time >= date.date_time}">{{date.display}}</span>
                            <span class="date--block grey-date" v-for="date in active_block.day_fillIn" @click="incrementBlock('up')" :class="{'active--day': date_select == date.date_time || date_range_select.start.date_time == date.date_time || date_range_select.end.date_time == date.date_time || isStart && date_range_select.start.date_time <= date.date_time && date_range_select.end.date_time >= date.date_time}">{{date.display}}</span>
                        </div>
                        <div class="date--group" v-if="active_block.next_block">
                            <span class="date--block grey-date" v-for="date in active_block.next_block.day_offset">{{date.display}}</span>
                            <span class="date--block" v-for="date in active_block.next_block.dates" :class="{'active--day': date_select == date.date_time || date_range_select.start.date_time == date.date_time || date_range_select.end.date_time == date.date_time || isStart && date_range_select.start.date_time <= date.date_time && date_range_select.end.date_time >= date.date_time}">{{date.display}}</span>
                            <span class="date--block grey-date" v-for="date in active_block.next_block.day_fillIn">{{date.display}}</span>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
    export default {
        props: ['type', 'set_language', 'format'],
        mounted() {
            this.setDate();
        },
        data: function () {
          return{
              timer: 0,
              activeReady: false,
              isTransition: false,
              isNext: false,
              isPrev: false,
              isStart: true,
              formats: ['YYYY-MM-DD'],
              active_block: '',
              day_display: '',
              date_select: '',
              date_range_select: {
                  start: {
                      'date_time': '',
                      'date_format': '',
                      date_display: {
                          'year': '',
                          'month': '',
                          'day': ''
                      }
                  },
                  end: {
                      'date_time': '',
                      'date_format': '',
                      date_display: {
                          'year': '',
                          'month': '',
                          'day': ''
                      }
                  }
              }
          }
        },
        methods:{
            setUniqueID: function () {
                console.log('yep');
            },
            incrementBlock: function (direction) {
                var moveTo, newActiveBlock;
                //determine the direction of the calendar swipe and new active_block
                if(direction == 'up'){
                    moveTo = 'isNext';
                    newActiveBlock = 'next_block';
                } else {
                    moveTo = 'isPrev';
                    newActiveBlock = 'prev_block';
                }
                  //activate the group container transition ease
                  this.isTransition = true;

                  //transition to the next month set of dates
                  this[moveTo] = true;

                  //after 1s delay set the next_block as the new active_block
                  var self = this;
                  clearTimeout(this.timer);
                  this.timer = setTimeout(function () {
                      //de-activate the group container transition ease
                      self.isTransition = false;

                      //reset the group transform back to start
                      self[moveTo] = false;

                      //make the block_switch
                      self.active_block = self.active_block[newActiveBlock];
                      self.active_block.next_block = self.calcNext('up', self.active_block.year, self.active_block.month);
                      self.active_block.prev_block = self.calcNext('down', self.active_block.year, self.active_block.month);
                  }, 1200);
            },
            setDate: function () {
                //get the current date and time
                var date = new Date();

                //set block function to create initial active_block object
                this.active_block = this.setBlock(date.getFullYear(), date.getMonth() + 1);

                //after new initial active_block set next and previous month blocks
                this.active_block.next_block = this.calcNext('up', this.active_block.year, this.active_block.month);
                this.active_block.prev_block = this.calcNext('down', this.active_block.year, this.active_block.month);

            },
            setBlock: function (year, month) {
                //return the block with the set amount of days
                var offset = this.dayOffSet(year,month);
                var dates = this.daysInMonth(year, month);
                return {
                    'day_offset': offset,
                    'day_fillIn': this.dayFillIn(year, month, dates, offset),
                    'year': year,
                    'month': month,
                    'days': [0,1,2,3,4,5,6],
                    'dates': dates
                }
            },
            calcNext: function (direction, year, month) {
                var max = 12, min = 1;
                //increment the month based on direction
                if(direction == 'up'){
                    //if month reaches the max reset to jan = 1;
                    if(month >= max){
                        month = 1;
                        //increment year when max
                        year++;
                    } else {
                        month++;
                    }
                } else {
                    //if month reaches the min reset to dec = 12;
                    if(month <= min){
                        month = 12;
                        //decrement year when min
                        year--;
                    } else {
                        month--;
                    }
                }
                var offset = this.dayOffSet(year,month);
                var dates = this.daysInMonth(year, month);
              return {
                  'day_offset': offset,
                  'day_fillIn': this.dayFillIn(year, month, dates, offset),
                  'year': year,
                  'month': month,
                  'days': [0,1,2,3,4,5,6],
                  'dates': dates
              }
            },
            dayFillIn: function (year, month, dates, offset) {
                var weeks = this.weekCount(year, month);
                    var total_days = (6 * 7);
                var toFill = total_days - (dates.length + offset.length);
                var fillDates = [];
                for(var i = 1; i <= toFill; i++){
                    fillDates.push(this.madeDay(year, month, i));
                }
                return fillDates;
            },
            weekCount: function(year, month_number) {

                // month_number is in the range 1..12

                var firstOfMonth = new Date(year, month_number, 1);
                var lastOfMonth = new Date(year, month_number, 0);

                var used = firstOfMonth.getDay() + lastOfMonth.getDate();

                return Math.ceil( used / 7);
            },
            selectDate: function (date) {
                var block = this.active_block;
                var day = date.display;
                if(this.type == 'date'){
                    this.day_display = day;
                    this.date_select = date.date_time;
                    console.log(this.formatDate(block.year, block.month, day));
                    return this.formatDate(block.year, block.month, day);
                }

                if(!this.isStart){
                    this.setDateRange('end', block.year, block.month, day, date.date_time);
                    console.log(this.date_range_select);
                    return this.isStart = true;
                }
                    this.clearRange();
                    this.isStart = false;
                this.setDateRange('start', block.year, block.month, day, date.date_time);

            },
            setDateRange: function (type, year, month, day, date_time) {
                this.date_range_select[type].date_display.day = day;
                this.date_range_select[type].date_display.month = month;
                this.date_range_select[type].date_display.year = year;
                this.date_range_select[type].date_format = this.formatDate(year, month, day);
                return this.date_range_select[type].date_time = date_time;
            },
            clearRange: function () {
                this.date_range_select.start.date_time = '';
              this.date_range_select.start.date_display.day = '';
                this.date_range_select.start.date_display.year = '';
                this.date_range_select.start.date_display.month = '';
                this.date_range_select.start.date_format = '';
                this.date_range_select.end.date_format = '';
                this.date_range_select.end.date_time = '';
                this.date_range_select.end.date_display.day = '';
                this.date_range_select.end.date_display.year = '';
                this.date_range_select.end.date_display.month = '';
            },
            dayOffSet: function (year,month) {
                //get the dates for the previous month
                var previous_month = month - 1;
                var previous_year = year;
                var min = 1;

                //find the offset for the current month by finding the weekday the first day of the week starts.
                var offset = new Date((new Date()).setFullYear(previous_year, previous_month, 1));
                var day_offset = offset.getDay();



                //if the pevious is below min set to dec = 12 and decrement year;
                if(previous_month < min){
                    previous_month = 12;
                    previous_year--;
                }

                //get the days in the previous month;
                var previous_days = this.daysInMonth(previous_year, previous_month);

                var offset_days = [];

                //get the days within the limit of the offset
                for(var i = previous_days.length; i > previous_days.length - day_offset; i--){
                    offset_days.push(this.madeDay(year, month, i));
                }

                return offset_days;
            },
            daysInMonth: function (year, month) {
                //get days in month using the year and month
                var count = new Date(year, month, 0).getDate(),
                        days = [];

                //push days to array to use in v-for loop
                for(var i = 0; i < count; i++){
                    days.push(this.madeDay(year, month, i + 1));
                }
                return days;
            },
            madeDay: function (year, month, day) {
                var date = new Date(year, month, day).getTime();
                return {
                    'date_time': date,
                    'display': day
                }
            },
            formatDate: function (year, month, day) {
                switch(this.format){
                    case "YYYY-MM-DD":
                        return year + '-' + month + '-' + day;
                        break;
                    default:
                        return;
                    break;
                }
            }
        },
        filters: {
            filterLanguageWeek: function (day,language) {
                var block = {
                    0: {
                        'EN': 'SUN',
                        'ES': 'DOM'
                    },
                    1: {
                        'EN': 'MON',
                        'ES': 'LUN'
                    },
                    2: {
                        'EN': 'TUE',
                        'ES': 'MAR'
                    },
                    3: {
                        'EN': 'WED',
                        'ES': 'MIE'
                    },
                    4: {
                        'EN': 'THUR',
                        'ES': 'JUE'
                    },
                    5: {
                        'EN': 'FRI',
                        'ES': 'VIE'
                    },
                    6: {
                        'EN': 'SAT',
                        'ES': 'SAB'
                    }
                };
                return block[day][language];
            },
            filterLanguageMonth: function (month,language) {
                var block = {
                    1: {
                        'EN': 'JANUARY',
                        'ES': 'ENERO'
                    },
                    2: {
                        'EN': 'FEBRUARY',
                        'ES': 'FEBRERO'
                    },
                    3: {
                        'EN': 'MARCH',
                        'ES': 'MARZO'
                    },
                    4: {
                        'EN': 'APRIL',
                        'ES': 'ABRIL'
                    },
                    5: {
                        'EN': 'MAY',
                        'ES': 'MAY0'
                    },
                    6: {
                        'EN': 'JUNE',
                        'ES': 'JUNIO'
                    },
                    7: {
                        'EN': 'JULY',
                        'ES': 'JULIO'
                    },
                    8: {
                        'EN': 'AUGUST',
                        'ES': 'AGOSTO'
                    },
                    9: {
                        'EN': 'SEPTEMBER',
                        'ES': 'SEPTIEMBRE'
                    },
                    10: {
                        'EN': 'OCTOBER',
                        'ES': 'OCTUBRE'
                    },
                    11: {
                        'EN': 'NOVEMBER',
                        'ES': 'NOVIEMBRE'
                    },
                    12: {
                        'EN': 'DECEMBER',
                        'ES': 'DICIEMBRE'
                    }
                };
                return block[month][language];
            }
        }
    }
</script>
