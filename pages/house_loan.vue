<template>
    <div class="house_loan">
        <nya-container title="房贷计算器">
            <nya-input v-model.trim="loanAmount" fullwidth type="number" label="贷款金额（万元）" autofocus placeholder="258" autocomplete="off" />
            <nya-input v-model.trim="loanTerm" fullwidth type="number" label="贷款期限（年）" autofocus placeholder="30" autocomplete="off" />
            <nya-input v-model.trim="loanRate" fullwidth type="number" label="贷款利率（%）" autofocus placeholder="5.45" autocomplete="off" />
            <nya-checkbox v-model="choosePrepayment" class="mb-15" label="提前还款" />
            <br>
            <div v-if="choosePrepayment">
                <client-only>
                    <label class="input-title">第一次还款时间</label>
                    <date-picker v-model="firstRepaymentDate" class="nya-input" :editable="false" placeholder="第一次还款时间" :default-value="firstRepaymentDateDefaultValue" value-type="format" :not-before="from" :not-after="to" />
                    <label class="input-title">预计提前还款时间</label>
                    <date-picker v-model="intendedRepaymentDate" class="nya-input" :editable="false" placeholder="预计提前还款时间" :default-value="intendedRepaymentDateDefaultValue" value-type="format" :not-before="from" :not-after="to" />
                    <nya-input v-model.trim="intendedRepaymentAmount" fullwidth type="number" label="预计提前还款金额（万元）" autofocus placeholder="30" autocomplete="off" />
                </client-only>
            </div>
            <button type="button" class="nya-btn" @click="calculate">
                {{ '开始计算' }}
            </button>
        </nya-container>
        <nya-container v-show="result" :title="`计算结果`">
            <pre>{{ result }}</pre>
        </nya-container>
        <nya-container title="说明">
            <ul class="nya-list">
                <li>还款方式默认为等额本息。</li>
            </ul>
        </nya-container>
    </div>
</template>

<script>
let DatePicker;
if (process.browser) {
    DatePicker = require('vue2-datepicker').default;
}
import dayjs from 'dayjs';
import Dynamic from '@/components/Dynamic';
export default {
    name: 'HouseLoan',
    components: {
        DatePicker,
        Dynamic
    },
    head() {
        return this.$store.state.currentTool.head;
    },
    data() {
        return {
            loanAmount: 258,
            loanTerm: 30,
            loanRate: 5.45,
            choosePrepayment: false,
            firstRepaymentDate: '',
            intendedRepaymentDate: '',
            intendedRepaymentAmount: 30,
            firstRepaymentDateDefaultValue: dayjs()
                .subtract(18, 'year')
                .format('YYYY-MM-DD'),
            intendedRepaymentDateDefaultValue: dayjs()
                .subtract(18, 'year')
                .format('YYYY-MM-DD'),
            from: dayjs()
                .subtract(78, 'year')
                .toDate(),
            to: dayjs()
                .subtract(1, 'year')
                .toDate(),
            result: '' //输出的结果
        };
    },
    watch: {
    },
    methods: {
        calculate() {
            this.result = 'abcd';
        }
    }
};
</script>

<style lang="scss">
.house_loan {
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
        font-size: 16px;
        margin-bottom: 10px;
        font-weight: bold;
    }
}
</style>
