<template>
    <div class="timestamp">
        <nya-container title="时间戳转换">
            <client-only>
                <label class="input-title">北京时间</label>
                <date-picker v-model="date" type="datetime" class="nya-input date-picker mt-15" format="YYYY-MM-DD HH:mm:ss" confirm :editable="true" placeholder="选择日期" :default-value="defaultValue" value-type="timestamp" @change="onDateChange"/>
            </client-only>
            <nya-input v-model.trim="timestampSec" fullwidth type="number" :label="`时间戳(秒)`" :placeholder="`时间戳(秒)`" autocomplete="off" @change="onTimestampSecChange"/>
            <nya-input v-model.trim="timestampMsec" fullwidth type="number" :label="`时间戳(毫秒)`" :placeholder="`时间戳(毫秒)`" autocomplete="off" @change="onTimestampMsecChange"/>
            <button type="button" class="nya-btn" @click="setCurrent">
                {{ '当前时间' }}
            </button>
            <button type="button" class="nya-btn" @click="clear">
                {{ '清空' }}
            </button>
        </nya-container>
    </div>
</template>

<script>
let DatePicker;
if (process.browser) {
    DatePicker = require('vue2-datepicker').default;
}
import dayjs from 'dayjs';
export default {
    name: 'Timestamp',
    head() {
        return this.$store.state.currentTool.head;
    },
    components: {
        DatePicker
    },
    data() {
        return {
            timestampSec: null,
            timestampMsec: null,
            defaultValue: dayjs().format('YYYY-MM-DD'),
            date: null,
        };
    },
    methods: {
        onDateChange(e) {
            if(this.date == null)
            {
                this.clear();
                return;
            }
            this.timestampSec = Math.floor(this.date / 1000);
            this.timestampMsec = this.date;
        },
        onTimestampSecChange(e) {
            this.timestampMsec = this.timestampSec * 1000;
            this.date = this.timestampMsec;
        },
        onTimestampMsecChange(e) {
            this.timestampSec = Math.floor(this.timestampMsec / 1000);
            this.date = parseInt(this.timestampMsec);
        },
        setCurrent() {
            this.timestampSec = dayjs().unix();
            this.timestampMsec = new Date().getTime();
            this.date = this.timestampMsec;
        },
        clear() {
            this.date = null;
            this.timestampSec = null;
            this.timestampMsec = null;
        }
    }
};
</script>

<style lang="scss">
.timestamp {
    .date-picker {
        width: 100%;
    }
    .nya-input {
        width: 100%;
        box-sizing: border-box;
        margin-bottom: 15px;
        .mx-input {
            border-radius: 0;
            height: 100%;
            box-sizing: border-box;
        }
    }
    .input-title {
        display: block;
        font-size: 18px;
        font-weight: bold;
    }
}
</style>
