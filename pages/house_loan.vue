<template>
    <div class="house_loan">
        <nya-container title="房贷计算器">
            <nya-input v-model.trim="loanAmount" fullwidth type="number" label="贷款金额（万元）" placeholder="258" autocomplete="off" />
            <nya-input v-model.trim="loanTerm" fullwidth type="number" label="贷款期限（年）" placeholder="30" autocomplete="off" />
            <nya-input v-model.trim="loanRate" fullwidth type="number" label="贷款利率（%）" placeholder="5.45" autocomplete="off" />
            <nya-checkbox v-model="choosePrepayment" class="mb-15" label="提前还款" />
            <br>
            <div v-if="choosePrepayment">
                <client-only>
                    <label class="input-title">第一次还款时间</label>
                    <date-picker v-model="firstRepaymentDate" class="nya-input" :editable="false" :placeholder="`${firstRepaymentDate}`" value-type="format" :not-before="from" :not-after="to" />
                    <label class="input-title">预计提前还款时间</label>
                    <date-picker v-model="intendedRepaymentDate" class="nya-input" :editable="false" :placeholder="`${intendedRepaymentDate}`" value-type="format" :not-before="from" :not-after="to" />
                    <nya-input v-model.trim="intendedRepaymentAmount" fullwidth type="number" label="预计提前还款金额（万元）" placeholder="30" autocomplete="off" />
                </client-only>
            </div>
            <button type="button" class="nya-btn" @click="calculate">
                {{ '开始计算' }}
            </button>
        </nya-container>
        <nya-container v-show="result" :title="`计算结果`">
            <p>累计支付利息：{{ `${result.totalInterest}` }}</p>
            <p>累计还款总额：{{ `${result.totalRepayment}` }}</p>
            <p>分期详情：</p>
            <table width="90%" align="center">
            <thead>
                <tr>
                    <th>期次</th>
                    <th>偿还本息(元)</th>
                    <th>偿还利息(元)</th>
                    <th>偿还本金(元)</th>
                    <th>剩余本金(元)</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="index of ${result.months}" :key="index">
                    <td>{{ index }}</td>
                    <td>{{ result.monthRepayment }}</td>
                    <td>{{ calcMonthInterest(index) }}</td>
                    <td>{{ result.monthRepayment - calcMonthInterest(index) }}</td>
                    <td>{{ calcRemainingAmount(index) }}</td>
                </tr>
            </tbody>
            </table>
        </nya-container>
        <nya-container v-show="false && result && choosePrepayment" :title="`提前还款结果`">
            <p>已还利息额：{{ `12345${result}` }}</p>
            <p>已还款总额：{{ `12345${result}` }}</p>
            <p>节省利息支出：{{ `12345${result}` }}</p>
            <p>该月还款额：{{ `12345${result}` }}</p>
            <p>分期详情：</p>
            <table width="90%" align="center">
            <thead>
                <tr>
                    <th>期次</th>
                    <th>偿还本息(元)</th>
                    <th>偿还利息(元)</th>
                    <th>偿还本金(元)</th>
                    <th>剩余本金(元)</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="(item) in vv">
                    <td>{{ item }}</td>
                    <td>{{ item }}</td>
                    <td>333</td>
                    <td>444</td>
                    <td>555</td>
                </tr>
            </tbody>
            </table>
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
            firstRepaymentDate: dayjs('2021-05-05').format('YYYY-MM-DD'),
            intendedRepaymentDate: dayjs('2023-05-05').format('YYYY-MM-DD'),
            intendedRepaymentAmount: 30,
            from: dayjs()
                .subtract(80, 'year')
                .toDate(),
            to: dayjs()
                .add(80, 'year')
                .toDate(),
            result: {} //输出的结果
        };
    },
    watch: {
    },
    methods: {
        calculate() {
            const monthRate = this.loanRate / 100 / 12;  // 月利率
            const months = this.loanTerm * 12;   // 还款月数
            const amount = this.loanAmount * 10000;  // 总贷款额
            const value = Math.pow(1 + monthRate, months);
            this.result.monthRepayment = amount * monthRate * value / (value - 1);  // 月均还款
            this.result.totalRepayment = months * amount * monthRate * value / (value - 1);  // 还款总额
            this.result.totalInterest = this.result.totalRepayment - amount;  // 利息总和
            this.result.monthRate = monthRate;
            this.result.months = months;
            this.result.amount = amount;
        },
        calcMonthInterest(index) {
            return (this.result.amount * this.result.monthRate - this.result.monthRepayment) * Math.pow(1 + this.result.monthRate, index - 1) + this.result.monthRepayment;
        },
        calcRemainingAmount(index) {
            repaymentAmount = 0;
            for (var i = 1; i <= this.result.months; i++)
            { 
                repaymentAmount += (this.result.monthRepayment - calcMonthInterest(index));
            }
            return this.result.amount - repaymentAmount;
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
        font-size: 18px;
        margin-bottom: 10px;
        font-weight: bold;
    }
    table tbody {
        display:block;
        height:195px;
        overflow-y:scroll;
        text-align: center;
    }
    table thead, tbody tr {
        display:table;
        width:100%;
        table-layout:fixed;
        border-bottom:1px solid;
    }
    tbody tr {
        border-bottom:1px dashed;
    }
    table thead {
        width: calc( 100% - 0.5em );
    }
}
</style>
