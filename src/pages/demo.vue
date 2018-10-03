<template>
    <div class="demo">
        <div>
            <h3>日期选择</h3>
            <datepicker dateType="date" :placeholder="placeholder" :startDate="startDate" :endDate="endDate" v-model="dateTime1"></datepicker>
        </div>
        <div>
            <h3>日期及时间选择</h3>
            <datepicker dateType="datetime" :placeholder="placeholder" :startDate="startDate" :endDate="endDate" v-model="dateTime2"></datepicker>
        </div>
    </div>
</template>

<script>
    import datepicker from '../components/datepicker.vue'

    export default {
        components: {
            datepicker,
        },
        data() {
            return {
                dateTime1: '',
                dateTime2: '',
                startDate: '2018-8-1',
                endDate: '2020-1-1',
            }
        },
        computed: {
            placeholder(){
                if(this.dateTime){
                    return this.dateTime
                }else{
                    return '请选择有效日期'
                }
            }
        },
        methods: {

        },
    }

</script>
<style scoped>
    .demo {
        width: 400px;
        margin: 50px auto;
        border: 1px solid #ccc;
        height: 240px;
        text-align: center;
    }

</style>
