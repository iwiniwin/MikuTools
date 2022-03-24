<template>
    <div class="cash_card_format">
        <nya-container title="现金卡格式转换">
            <nya-input
                v-model="n"
                fullwidth
                class="mb-15"
                type="file"
                accept=".xls,.xlsx"
                label="请选择要转换的现金卡文档"
                placeholder="点击这里上传文件"
                @change="readExcel($event)"
            />
        </nya-container>
        <nya-container v-show="result" title="转换成功">
            <pre>{{ result }}</pre>
        </nya-container>
    </div>
</template>
<script>
import XLSX from "xlsx";

export default {
    data: () => ({
        n: '',
        result: '' //输出的结果
    }),
    methods: {
        readExcel(e) {
            // 读取表格文件
            const files = e.target.files;
            if (files.length <= 0) {
                return false;
            } else if (!/\.(xls|xlsx)$/.test(files[0].name.toLowerCase())) {
                this.$message({
                    message: "上传格式不正确，请上传xls或者xlsx格式",
                    type: "warning"
                });
                return false;
            }

            const fileReader = new FileReader();
            fileReader.onload = ev => {
                try {
                    const data = ev.target.result;
                    const workbook = XLSX.read(data, {
                        type: "binary"
                    });
                    const wsname = workbook.SheetNames[0]; //取第一张表
                    const ws = XLSX.utils.sheet_to_json(workbook.Sheets[wsname]); //生成json表格内容
                    console.log(ws);
                } catch (e) {
                    return false;
                }
            };
            fileReader.readAsBinaryString(files[0]);
        }
    }
};
</script>