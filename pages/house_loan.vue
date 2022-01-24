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
                    <date-picker v-model="firstRepaymentDate" class="nya-input" :editable="false" :placeholder="`${firstRepaymentDate}`" value-type="format" format="YYYY-MM" type="month" :not-before="from" :not-after="to" />
                    <label class="input-title">预计提前还款时间</label>
                    <date-picker v-model="intendedRepaymentDate" class="nya-input" :editable="false" :placeholder="`${intendedRepaymentDate}`" value-type="format" format="YYYY-MM" type="month" :not-before="from" :not-after="to" />
                    <nya-input v-model.trim="intendedRepaymentAmount" fullwidth type="number" label="预计提前还款金额（万元）" placeholder="30" autocomplete="off" />
                </client-only>
            </div>
            <button type="button" class="nya-btn" @click="calculate">
                {{ '开始计算' }}
            </button>
        </nya-container>
        <nya-container v-show="result.calculated" :title="`计算结果`">
            <p>累计支付利息：{{ `${toFixed2(result.loanInfo.totalInterest)} 元` }}</p>
            <p>累计还款总额：{{ `${toFixed2(result.loanInfo.totalRepayment)} 元` }}</p>
            <p>分期详情：</p>
            <table class="detail-table" width="90%" align="center">
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
                <tr v-for="index of result.loanInfo.months" :key="index">
                    <td>{{ `${index}` }}</td>
                    <td>{{ `${toFixed2(result.loanInfo.monthRepayment)}` }}</td>
                    <td>{{ `${toFixed2(calcMonthInterest(index, result.loanInfo))}` }}</td>
                    <td>{{ `${toFixed2(result.loanInfo.monthRepayment - calcMonthInterest(index, result.loanInfo))}` }}</td>
                    <td>{{ `${toFixed2(calcRemainingAmount(index, result.loanInfo))}` }}</td>
                </tr>
            </tbody>
            </table>
        </nya-container>
        <nya-container v-show="choosePrepayment && result.prepayment.calculated" :title="`提前还款结果`">
            <p>已还利息额：{{ `${toFixed2(result.prepayment.paidInterest)} 元` }}</p>
            <p>已还款总额：{{ `${toFixed2(result.loanInfo.monthRepayment * result.prepayment.pastMonths)} 元` }}</p>
            <p>节省利息支出：{{ `${toFixed2(result.loanInfo.monthRepayment * result.prepayment.pastMonths)} 元` }}</p>
            <p>该月还款额：{{ `${toFixed2(result.loanInfo.monthRepayment * result.prepayment.pastMonths)} 元` }}</p>
            <p>分期详情：</p>
            <table class="detail-table" width="90%" align="center">
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
                <tr v-for="index of result.prepayment.loanInfo.months" :key="index">
                    <td>{{ `${index}` }}</td>
                    <td>{{ `${toFixed2(result.prepayment.loanInfo.monthRepayment)}` }}</td>
                    <td>{{ `${toFixed2(calcMonthInterest(index, result.prepayment.loanInfo))}` }}</td>
                    <td>{{ `${toFixed2(result.prepayment.loanInfo.monthRepayment - calcMonthInterest(index, result.prepayment.loanInfo))}` }}</td>
                    <td>{{ `${toFixed2(calcRemainingAmount(index, result.prepayment.loanInfo))}` }}</td>
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
            firstRepaymentDate: dayjs('2021-05-01').format('YYYY-MM'),
            intendedRepaymentDate: dayjs('2023-05-01').format('YYYY-MM'),
            intendedRepaymentAmount: 30,
            from: dayjs()
                .subtract(80, 'year')
                .toDate(),
            to: dayjs()
                .add(80, 'year')
                .toDate(),
            result: {
                calculated: false,
                loanInfo: {},
                prepayment: {
                    calculated: false,
                    loanInfo: {}
                }
            } //输出的结果
        };
    },
    watch: {
    },
    methods: {
        calculateLoanInfo(yearRate, term, amount) {
            const loanInfo = {
                monthRate: yearRate / 100 / 12,  // 月利率
                months: term * 12, // 还款月数
                amount: amount,  // 总贷款额
                monthRepayment: 0,
                totalRepayment: 0,
                totalInterest: 0
            };
            const value = Math.pow(1 + loanInfo.monthRate, loanInfo.months);
            loanInfo.monthRepayment = loanInfo.amount * loanInfo.monthRate * value / (value - 1);  // 月均还款
            loanInfo.totalRepayment = loanInfo.months * loanInfo.amount * loanInfo.monthRate * value / (value - 1);  // 还款总额
            loanInfo.totalInterest = loanInfo.totalRepayment - loanInfo.amount;  // 利息总和
            return loanInfo;
        },
        calculate() {
            this.result.loanInfo = this.calculateLoanInfo(this.loanRate, this.loanTerm, this.loanAmount * 10000);
            this.result.calculated = true;
            if(this.choosePrepayment) {
                const firstDate = dayjs(this.firstRepaymentDate);
                const intendedDate = dayjs(this.intendedRepaymentDate);
                const pastMonths = intendedDate.diff(firstDate, 'month'); // 已经过去的时间(月)
                const intendedAmount = this.intendedRepaymentAmount * 10000;  // 提前还款额
                const remainingAmount = this.calcRemainingAmount(pastMonths, this.result.loanInfo);
                const amount = remainingAmount - intendedAmount;  // 剩余贷款额

                this.result.prepayment.paidInterest = this.result.loanInfo.amount - remainingAmount; //已还利息
                this.result.prepayment.pastMonths = pastMonths;
                this.result.prepayment.loanInfo = this.calculateLoanInfo(this.loanRate, this.loanTerm, amount);
                this.result.prepayment.calculated = true;
            }
        },
        calcMonthInterest(index, loanInfo) {
            return (loanInfo.amount * loanInfo.monthRate - loanInfo.monthRepayment) * Math.pow(1 + loanInfo.monthRate, index - 1) + loanInfo.monthRepayment;
        },
        calcRemainingAmount(index, loanInfo) {
            var repaymentAmount = 0;
            for (var i = 1; i <= index; i++)
            { 
                repaymentAmount += (loanInfo.monthRepayment - this.calcMonthInterest(i, loanInfo));
            }
            return loanInfo.amount - repaymentAmount;
        },
        toFixed2(num) {
            if(typeof(num)=='undefined'){
                return num
            }else{
                return num.toFixed(2)
            }
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
    .detail-table tbody {
        display:block;
        height:195px;
        overflow-y:scroll;
        text-align: center;
    }
    .detail-table thead, .detail-table tbody tr {
        display:table;
        width:100%;
        table-layout:fixed;
        border-bottom:1px solid;
    }
    .detail-table tbody tr {
        border-bottom:1px dashed;
    }
    .detail-table thead {
        width: calc( 100% - 0.5em );
    }
}
</style>
