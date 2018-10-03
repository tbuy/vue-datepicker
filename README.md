# vue-datepicker

forked from hilongjw/vue-datepicker

该分支基于vue进行工程化并重构部分代码（含详细注释）去掉了momentjs 使用原生方法计算时间

# Demo

The demo page is [HERE](https://tbuy.github.io/vue-datepicker/dist/#/).

# API



名称 | 类型 | 默认值 | 是否必填 | 描述
---|--- |--- |--- | ---
dateType | String | "date" | 否 | 传入的时间类型（date：日期；dateTime：含时间的日期）
startDate | String | "1900-1-1" | 否 | 日期范围的起始时间
endDate | String | "2099-1-1" | 否 | 日期范围的结束时间
placeholder | String | "请选择日期" | 否 | 占位文字
week | Array | ["周一", "周二", "周三", "周四", "周五", "周六", "周日"] | 否 | 周
month | Array | ["一月","二月","三月","四月","五月","六月","七月","八月","九月","十月","十一月","十二月"] | 否 | 月
buttons | Object |  "{'cancel': '取消', 'ok': '确认'}"  | 否 | 自定义确认或者取消按钮文字
