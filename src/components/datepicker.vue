<template>
    <div class="cov-vue-date" :class="option.wrapperClass ? option.wrapperClass : {}">
        <!-- 日期表单 -->
        <div class="datepickbox" @click="showCheck">
            <input type="text" title="input date" class="cov-datepicker" readonly="readonly" :placeholder="placeholder" v-model="datetimes"
                @click="showCheck" @focus="showCheck" :style="option.inputStyle ? option.inputStyle : {}" :class="option.inputClass ? option.inputClass : {}"
            />
        </div>
        <!-- 选择日期弹层 -->
        <div class="datepicker-overlay" v-if="showInfo.check" v-bind:style="{'background' : option.overlayOpacity ? 'rgba(0,0,0,'+ option.overlayOpacity  +')' : 'rgba(0,0,0,0.5)'}">
            <div class="cov-date-body" :style="{'background-color': option.color ? option.color.header : '#3f51b5'}">
                <!-- top -->
                <div class="cov-date-monthly">
                    <div class="cov-date-previous" @click="nextMonth('pre')">«</div>
                    <div class="cov-date-caption" :style="{'color': option.color ? option.color.headerText : '#fff'}">
                        <span @click="showYear"><small>{{checked.year}}</small></span>
                        <br>
                        <span @click="showMonth">{{displayMonth}}</span>
                    </div>
                    <div class="cov-date-next" @click="nextMonth('next')">»</div>
                </div>
                <!-- middle -->
                <div class="cov-date-box" v-if="showInfo.day">
                    <div class="cov-picker-box">
                        <div class="week">
                            <ul>
                                <li v-for="weekie in library.week" :key="weekie">{{weekie}}</li>
                            </ul>
                        </div>
                        <div class="day" v-for="(day, index) in dayList" :key="index" @click="checkDay(day)" :class="{'checked':day.checked,'unavailable':day.unavailable}"
                            :style="day.checked ? (option.color && option.color.checkedDay ? { background: option.color.checkedDay } : { background: '#3F51B5' }) : {}">
                            {{day.value}}
                        </div>
                    </div>
                </div>
                <div class="cov-date-box list-box" v-if="showInfo.year">
                    <div class="cov-picker-box date-list" id="yearList">
                        <div class="date-item" v-for="yearItem in library.year" track-by="$index" :key="yearItem" @click="setYear(yearItem)">{{yearItem}}
                        </div>
                    </div>
                </div>
                <div class="cov-date-box list-box" v-if="showInfo.month">
                    <div class="cov-picker-box date-list">
                        <div class="date-item" v-for="(monthItem,index) in library.month" track-by="$index" :key="monthItem" @click="setMonth(index + 1)">{{monthItem}}
                        </div>
                    </div>
                </div>
                <div class="cov-date-box list-box" v-if="showInfo.hour">
                    <div class="cov-picker-box date-list">
                        <div class="watch-box">
                            <div class="hour-box">
                                <div class="mui-pciker-rule mui-pciker-rule-ft"></div>
                                <ul>
                                    <li class="hour-item" v-for="hitem in hours" :key="hitem.value" @click="setTime('hour', hitem, hours)" :class="{'active':hitem.checked}">
                                        {{hitem.value}}
                                    </li>
                                </ul>
                            </div>
                            <div class="min-box">
                                <div class="mui-pciker-rule mui-pciker-rule-ft"></div>
                                <div class="min-item" v-for="mitem in mins" :key="mitem.value" @click="setTime('min',mitem, mins)" :class="{'active':mitem.checked}">
                                    {{mitem.value}}
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
                <!-- bottom -->
                <div class="button-box">
                    <span @click="picked('cancel')">{{buttons && buttons.cancel ? buttons.cancel : '取消'}}</span>
                    <span @click="picked">{{buttons && buttons.ok ? buttons.ok : '确定'}}</span>
                </div>
            </div>
        </div>
    </div>
</template>
<script>
    export default {
        props: {
            //双向绑定的值
            value: {
                type: String,
                default: ''
            },
            //传入的时间类型
            dateType: {
                type: String,
                default: "date"
            },
            //日期范围的起始时间
            startDate: {
                type: String,
                default: "1900-1-1"
            },
            //日期范围的结束时间
            endDate: {
                type: String,
                default: "2099-1-1"
            },
            //占位文字
            placeholder: {
                type: String,
                default: "请选择日期"
            },
            //周
            week: {
                type: Array,
            },
            //月
            month: {
                type: Array,
            },
            //自定义确认或者取消按钮文字
            buttons: {
                type: Object,
                default: () => {}
            },
        },
        data() {
            // 小时
            function hours() {
                let list = [];
                let hour = 24;
                while (hour > 0) {
                    hour--;
                    list.push({
                        checked: false,
                        // 小于10的前一位补0
                        value: hour < 10 ? "0" + hour : hour
                    });
                }
                return list;
            }
            // 分钟
            function mins() {
                let list = [];
                let min = 60;
                while (min > 0) {
                    min--;
                    list.push({
                        checked: false,
                        // 小于10的前一位补0
                        value: min < 10 ? "0" + min : min
                    });
                }
                return list;
            }

            return {
                // 传入的时间
                datetimes: this.value,
                // 配置项
                option: {
                    // 传入的时间类型
                    type: this.dateType == "date" ? "day" : "min",
                    // 日期格式化
                    format: this.dateType == "date" ? "yyyy-MM-dd" : "yyyy-MM-dd hh:mm",
                    // 输出框样式
                    inputStyle: {
                        color: "#000",
                        display: "block",
                        width: "100%",
                        "box-sizing": "border-box",
                        padding: "5px",
                        border: "solid 1px #C6C6C6"
                    },
                    //弹出背景透明度
                    overlayOpacity: 0.5,
                },
                // 小时
                hours: hours(),
                // 分钟
                mins: mins(),
                //middle显示哪种选择模块
                showInfo: {
                    hour: false,
                    day: false,
                    month: false,
                    year: false,
                    check: false
                },
                //展示给用户选中的月份
                displayMonth: "",
                //弹层显示的文字
                library: {
                    // 周
                    week: this.week ? this.week : ["周一", "周二", "周三", "周四", "周五", "周六", "周日"],
                    // 月
                    month: this.month ? this.month : [
                        "一月",
                        "二月",
                        "三月",
                        "四月",
                        "五月",
                        "六月",
                        "七月",
                        "八月",
                        "九月",
                        "十月",
                        "十一月",
                        "十二月"
                    ],
                    // 年
                    year: []
                },
                //选中的参数
                checked: {
                    //上一次选中的日期
                    oldtime: "",
                    //年
                    year: "",
                    //月
                    month: "",
                    //日
                    day: "",
                    //时
                    hour: "",
                    //分
                    min: "",
                    //当前选中日期
                    date: this.value ? new Date(this.value) : new Date()
                },
                // 日期列表（上月，当月，下月）
                dayList: [],
            };
        },
        watch: {
            datetimes() {
                if (this.value != this.datetimes) {
                    this.$emit('input', this.datetimes)
                }
            },
        },
        methods: {
            //日期格式化
            formatDate(date, fmt) {
                var o = {
                    "M+": date.getMonth() + 1, //月份
                    "d+": date.getDate(), //日
                    "h+": date.getHours(), //小时
                    "m+": date.getMinutes(), //分
                    "s+": date.getSeconds(), //秒
                    "q+": Math.floor((date.getMonth() + 3) / 3), //季度
                    "S": date.getMilliseconds() //毫秒
                };
                if (/(y+)/.test(fmt))
                    fmt = fmt.replace(RegExp.$1, (date.getFullYear() + "").substr(4 - RegExp.$1.length));
                for (var k in o)
                    if (new RegExp("(" + k + ")").test(fmt))
                        fmt = fmt.replace(RegExp.$1, (RegExp.$1.length == 1) ? (o[k]) : (("00" + o[k]).substr(("" + o[k])
                            .length)));
                return fmt;
            },
            //间隔
            calcInterval(day1, day2) {
                var strDateStart = this.formatDate(day1, 'yyyy-MM-dd')
                var strDateEnd = this.formatDate(day2, 'yyyy-MM-dd')

                //日期分隔符
                var strSeparator = "-";
                var oDate1;
                var oDate2;
                var iDays;
                oDate1 = strDateStart.split(strSeparator);
                oDate2 = strDateEnd.split(strSeparator);
                var strDateS = new Date(oDate1[0], oDate1[1] - 1, oDate1[2]);
                var strDateE = new Date(oDate2[0], oDate2[1] - 1, oDate2[2]);

                return parseInt((strDateS - strDateE) / 1000 / 60 / 60 / 24)
            },
            //过滤数字 10以内数字以0开头
            numberFilter(n) {
                n = Math.floor(n);
                return n < 10 ? "0" + n : n;
            },
            // 切换月份
            nextMonth(type) {
                //当前选中日期对应的月份
                var _month = this.checked.date.getMonth() + 1;
                //限制
                var _startYear = this.startDate.slice(0, 4);
                var _startMonth = this.startDate.slice(5, 7);
                if (_startMonth.indexOf('-') == -1 || _startMonth.indexOf('/') == -1) {
                    _startMonth = _startMonth.slice(0, 1)
                }
                var _endYear = this.endDate.slice(0, 4);
                var _endMonth = this.endDate.slice(5, 7);
                if (_endMonth.indexOf('-') == -1 || _endMonth.indexOf('/') == -1) {
                    _endMonth = _endMonth.slice(0, 1)
                }

                if (type === 'next') {
                    if (this.checked.year > _endYear || this.checked.year == _endYear && this.checked.month == _endMonth) {
                        return false;
                    } else {
                        this.setMonth(_month + 1)

                    }
                } else {
                    if (this.checked.year < _startYear || this.checked.year == _startYear && this.checked.month == _startMonth) {
                        return false;
                    } else {
                        this.setMonth(_month - 1)
                    }
                }

            },
            //弹层勾选的时间
            showDay() {
                //设置当前日期
                this.showOne("day");
                this.checked.year = this.checked.date.getFullYear();
                this.checked.month = this.checked.date.getMonth() + 1;
                this.checked.day = this.checked.date.getDate();
                //设置当前月
                this.displayMonth = this.library.month[this.checked.date.getMonth()];
                //弹层中显示的全部日期
                var _days = [];
                //当前月份第一天是星期几 (0-6) 0是周日7
                var _week = (new Date(this.checked.year, this.checked.month - 1, 1)).getDay() ? (new Date(this.checked.year,
                    this.checked.month - 1, 1)).getDay() : 7;
                //当前月有几天
                var _monthDays = (new Date(this.checked.year, this.checked.month, 0)).getDate();

                for (var i = 1; i <= _monthDays; ++i) {
                    _days.push({
                        value: i,
                        inMonth: true, //是否当月
                        unavailable: false, //是否可控制
                        checked: false, //是否选择
                    });

                }

                for (var i = 0; i < _week - 1; i++) {
                    //上月可显示时间插入组件中显示
                    var _previousDay = {
                        value: (new Date(this.checked.year, this.checked.month - 1, 0)).getDate() - i, //上月时间
                        inMonth: false,
                        action: "previous",
                        unavailable: false,
                        checked: false,
                    };
                    _days.unshift(_previousDay);
                }
                //下月可显示时间插入组件中显示
                var _nextDayLength = 35 - _days.length;
                for (var i = 1; i < _nextDayLength + 1; i++) {
                    var _nextDay = {
                        value: i,
                        inMonth: false,
                        action: "next",
                        unavailable: false,
                        checked: false,
                    };
                    _days.push(_nextDay);
                }
                this.dayList = _days;

                //设置选中日期
                this.dayList.map(item => {

                    this.limitFromTo(item)

                    if (!item.unavailable && item.inMonth && item.value == this.checked.day) {
                        item.checked = true;
                    }

                })
            },
            //根据起始日期配置做限制
            limitFromTo(day) {
                var _start = new Date(this.startDate);
                var _end = new Date(this.endDate);
                var _day;

                if (day.action == 'previous') {
                    _day = new Date(this.checked.year, this.checked.month - 1 - 1, day.value);
                } else if (day.action == 'next') {
                    _day = new Date(this.checked.year, this.checked.month + 1 - 1, day.value);
                } else {
                    _day = new Date(this.checked.year, this.checked.month - 1, day.value);
                }
                if (this.calcInterval(_day, _start) > 0 && this.calcInterval(_day, _end) <= 0) {
                    day.unavailable = false;
                } else {
                    day.unavailable = true;
                }

                return day;
            },
            //选择日期 day in dayList
            checkDay(day) {
                //判断日期是否可选
                if (day.unavailable || day.value === "") {
                    return false;
                }
                //如果日期不是当月 切换至所在月份
                if (!day.inMonth) {
                    this.nextMonth(day.action);
                }
                //如果是day或者min类型 选中当前日期
                if (this.option.type === "day" || this.option.type === "min") {

                    this.dayList.map(item => {
                        item.checked = false;
                    });

                    this.checked.day = day.value;
                    day.checked = true;

                }

                //day类型 选中后直接关闭 min类型 选中后打开时间选项 @hour
                switch (this.option.type) {
                    case "day":
                        this.picked();
                        break;
                    case "min":
                        this.showOne("hour");
                        this.shiftActTime();
                        break;
                }
            },
            //展示选择年
            showYear() {
                var _year = this.checked.date.getFullYear();
                //startDate和endDate对年份的限制
                if (this.endDate.slice(0, 4) - _year < 4) {
                    _year = _year - (4 - (this.endDate.slice(0, 4) - _year))
                }
                this.library.year = [];

                for (var i = this.startDate.slice(0, 4); i < _year + 5; ++i) {
                    this.library.year.push(i);
                }
                this.showOne("year");
                //向上滚动增加截止年份
                this.$nextTick(() => {
                    var _listDom = document.getElementById("yearList");
                    _listDom.scrollTop = _listDom.scrollHeight - 100;
                    this.addYear();
                });
            },
            //根据类型在middle部分显示对应的模块
            showOne(type) {
                switch (type) {
                    case "year":
                        this.showInfo.hour = false;
                        this.showInfo.day = false;
                        this.showInfo.year = true;
                        this.showInfo.month = false;
                        break;
                    case "month":
                        this.showInfo.hour = false;
                        this.showInfo.day = false;
                        this.showInfo.year = false;
                        this.showInfo.month = true;
                        break;
                    case "day":
                        this.showInfo.hour = false;
                        this.showInfo.day = true;
                        this.showInfo.year = false;
                        this.showInfo.month = false;
                        break;
                    case "hour":
                        this.showInfo.hour = true;
                        this.showInfo.day = false;
                        this.showInfo.year = false;
                        this.showInfo.month = false;
                        break;
                    default:
                        this.showInfo.day = true;
                        this.showInfo.year = false;
                        this.showInfo.month = false;
                        this.showInfo.hour = false;
                }
            },
            //展示选择月
            showMonth() {
                this.showOne("month");
            },
            //添加年份
            addYear() {
                var _listDom = document.getElementById("yearList");
                _listDom.addEventListener("scroll", e => {
                    if (_listDom.scrollTop < _listDom.scrollHeight - 100) {
                        var _lastYear = this.library.year[this.library.year.length - 1];
                        if (_lastYear < this.endDate.slice(0, 4)) {
                            this.library.year.push(_lastYear + 1);
                        }
                    }
                }, false);
            },
            //设置年
            setYear(year) {
                setTimeout(() => {
                    // 存入修改之后的日期
                    this.checked.date = new Date(year, this.checked.month - 1, this.checked.day)
                    this.showDay();
                }, 20)
            },
            //设置月
            setMonth(month) {
                this.displayMonth = this.library.month[month - 1]
                this.checked.date = new Date(this.checked.year, month - 1, this.checked.day)
                this.showDay();
            },
            //显示选择日期弹层
            showCheck() {
                var date = new Date();
                //添加选择时间页面的默认小时（为当前小时数+1小时）
                this.hours.map(item => {
                    if (item.value == date.getHours()) {
                        item.checked = true;
                        this.checked.hour = new Date().getHours();
                    } else {
                        item.checked = false;
                    }
                });
                //添加选择时间页面的默认分钟（默认为xx小时：00分钟）
                this.mins.map(item => {
                    if (item.value == date.getMinutes()) {
                        item.checked = true;
                        this.checked.min = new Date().getMinutes();
                    } else {
                        item.checked = false;
                    }
                });
                //如果没有时间 弹层勾选当前日期
                if (this.datetimes === "") {
                    this.showDay();
                } else {
                    if (this.option.type === "day" || this.option.type === "min") {
                        this.checked.oldtime = this.datetimes;
                        this.showDay();
                    }
                }
                //显示弹出
                this.showInfo.check = true;
            },
            //设置时间（hour,min）
            setTime(type, obj, list) {
                list.forEach(item => {
                    item.checked = false;
                    if (item.value === obj.value) {
                        item.checked = true;
                        this.checked[type] = item.value;
                    }
                })
            },
            //取消或者确认按钮
            picked(type) {
                var _time;
                if (type == "cancel") {
                    this.$emit("cancel", this.datetimes);
                } else if (this.option.type === "day") {
                    _time = `${this.checked.year}-${this.checked.month}-${this.checked.day}`
                    this.checked.date = new Date(_time);
                    this.datetimes = this.formatDate(this.checked.date, this.option.format);
                } else if (this.option.type === "min") {
                    _time =
                        `${this.checked.year}-${this.checked.month}-${this.checked.day} ${this.checked.hour}:${this.checked.min}`
                    this.checked.date = new Date(_time);
                    this.datetimes = this.formatDate(this.checked.date, this.option.format);
                }
                this.showInfo.check = false;
                this.$emit("change", this.datetimes);
            },
            //选择时间的滚动动画
            shiftActTime() {
                this.$nextTick(() => {
                    if (!document.querySelector(".hour-item.active")) {
                        return false;
                    }
                    document.querySelector(".hour-box").scrollTop = (document.querySelector(".hour-item.active")
                        .offsetTop || 0) - 250;
                    document.querySelector(".min-box").scrollTop = (document.querySelector(".min-item.active").offsetTop ||
                        0) - 250;
                });
            }
        },
    };

</script>

<style scoped>
    .datepicker-overlay {
        position: fixed;
        width: 100%;
        height: 100%;
        z-index: 998;
        top: 0;
        left: 0;
        overflow: hidden;
        -webkit-animation: fadein 0.5s;
        /* Safari, Chrome and Opera > 12.1 */
        -moz-animation: fadein 0.5s;
        /* Firefox < 16 */
        -ms-animation: fadein 0.5s;
        /* Internet Explorer */
        -o-animation: fadein 0.5s;
        /* Opera < 12.1 */
        animation: fadein 0.5s;
    }

    @keyframes fadein {
        from {
            opacity: 0;
        }
        to {
            opacity: 1;
        }
    }

    /* Firefox < 16 */

    @-moz-keyframes fadein {
        from {
            opacity: 0;
        }
        to {
            opacity: 1;
        }
    }

    /* Safari, Chrome and Opera > 12.1 */

    @-webkit-keyframes fadein {
        from {
            opacity: 0;
        }
        to {
            opacity: 1;
        }
    }

    /* Internet Explorer */

    @-ms-keyframes fadein {
        from {
            opacity: 0;
        }
        to {
            opacity: 1;
        }
    }

    /* Opera < 12.1 */

    @-o-keyframes fadein {
        from {
            opacity: 0;
        }
        to {
            opacity: 1;
        }
    }

    .cov-date-body {
        background: #3F51B5;
        overflow: hidden;
        font-size: 16px;
        font-family: 'Roboto';
        font-weight: 400;
        position: fixed;
        display: block;
        width: 400px;
        max-width: 100%;
        z-index: 999;
        top: 50%;
        left: 50%;
        -webkit-transform: translate(-50%, -50%);
        -ms-transform: translate(-50%, -50%);
        transform: translate(-50%, -50%);
        box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.2);
    }

    .cov-picker-box {
        background: #fff;
        width: 100%;
        display: inline-block;
        padding: 25px;
        box-sizing: border-box !important;
        -moz-box-sizing: border-box !important;
        -webkit-box-sizing: border-box !important;
        -ms-box-sizing: border-box !important;
        width: 400px;
        max-width: 100%;
        height: 280px;
        text-align: start !important;
    }

    .cov-picker-box td {
        height: 34px;
        width: 34px;
        padding: 0;
        line-height: 34px;
        color: #000;
        background: #fff;
        text-align: center;
        cursor: pointer;
    }

    .cov-picker-box td:hover {
        background: #E6E6E6;
    }

    table {
        border-collapse: collapse;
        border-spacing: 0;
        width: 100%;
    }

    .day {
        width: 14.2857143%;
        display: inline-block;
        text-align: center;
        cursor: pointer;
        height: 34px;
        padding: 0;
        line-height: 34px;
        color: #000;
        background: #fff;
        vertical-align: middle;
    }

    .week ul {
        margin: 0 0 8px;
        padding: 0;
        list-style: none;
    }

    .week ul li {
        width: 14.2%;
        display: inline-block;
        text-align: center;
        background: transparent;
        color: #000;
        font-weight: bold;
    }

    .passive-day {
        color: #bbb;
    }

    .checked {
        background: rgb(63, 81, 181);
        color: #FFF !important;
        border-radius: 3px;
    }

    .unavailable {
        color: #ccc;
        cursor: not-allowed;
    }

    .cov-date-monthly {
        height: 150px;
    }

    .cov-date-monthly>div {
        display: inline-block;
        padding: 0;
        margin: 0;
        vertical-align: middle;
        color: #fff;
        height: 150px;
        float: left;
        text-align: center;
        cursor: pointer;
    }

    .cov-date-previous,
    .cov-date-next {
        position: relative;
        width: 20% !important;
        text-indent: -300px;
        overflow: hidden;
        color: #fff;
    }

    .cov-date-caption {
        width: 60%;
        padding: 50px 0 !important;
        box-sizing: border-box;
        font-size: 24px;
    }

    .cov-date-caption span:hover {
        color: rgba(255, 255, 255, 0.7);
    }

    .cov-date-previous:hover,
    .cov-date-next:hover {
        background: rgba(255, 255, 255, 0.1);
    }

    .day:hover {
        background: #EAEAEA;
    }

    .unavailable:hover {
        background: none;
    }

    .checked:hover {
        background: #FF4F8E;
    }

    .cov-date-next::before,
    .cov-date-previous::before {
        width: 20px;
        height: 2px;
        text-align: center;
        position: absolute;
        background: #fff;
        top: 50%;
        margin-top: -7px;
        margin-left: -7px;
        left: 50%;
        line-height: 0;
        content: '';
        -webkit-transform: rotate(45deg);
        -moz-transform: rotate(45deg);
        transform: rotate(45deg);
    }

    .cov-date-next::after,
    .cov-date-previous::after {
        width: 20px;
        height: 2px;
        text-align: center;
        position: absolute;
        background: #fff;
        margin-top: 6px;
        margin-left: -7px;
        top: 50%;
        left: 50%;
        line-height: 0;
        content: '';
        -webkit-transform: rotate(-45deg);
        -moz-transform: rotate(-45deg);
        transform: rotate(-45deg);
    }

    .cov-date-previous::after {
        -webkit-transform: rotate(45deg);
        -moz-transform: rotate(45deg);
        transform: rotate(45deg);
    }

    .cov-date-previous::before {
        -webkit-transform: rotate(-45deg);
        -moz-transform: rotate(-45deg);
        transform: rotate(-45deg);
    }

    .date-item {
        text-align: center;
        font-size: 20px;
        padding: 10px 0;
        cursor: pointer;
    }

    .date-item:hover {
        background: #e0e0e0;
    }

    .date-list {
        overflow: auto;
        vertical-align: top;
        padding: 0;
    }

    .cov-vue-date {
        display: inline-block;
        color: #5D5D5D;
    }

    .button-box {
        background: #fff;
        vertical-align: top;
        height: 50px;
        line-height: 50px;
        text-align: right;
        padding-right: 20px;
    }

    .button-box span {
        cursor: pointer;
        padding: 10px 20px;
    }

    .watch-box {
        height: 100%;
        overflow: hidden;
    }

    .hour-box,
    .min-box {
        display: inline-block;
        width: 50%;
        text-align: center;
        height: 100%;
        overflow: auto;
        float: left;
    }

    .hour-box ul,
    .min-box ul {
        list-style: none;
        margin: 0;
        padding: 0;
    }

    .hour-item,
    .min-item {
        padding: 10px;
        font-size: 36px;
        cursor: pointer;
    }

    .hour-item:hover,
    .min-item:hover {
        background: #E3E3E3;
    }

    .hour-box .active,
    .min-box .active {
        background: rgb(63, 81, 181);
        color: #FFF !important;
    }

    ::-webkit-scrollbar {
        width: 2px;
    }

    ::-webkit-scrollbar-track {
        background: #E3E3E3;
    }

    ::-webkit-scrollbar-thumb {
        background: #C1C1C1;
        border-radius: 2px;
    }

</style>
