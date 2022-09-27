<template>
    <div class="cash_card_format">
        <nya-container title="现金卡格式转换">
            <nya-input v-model="content" fullwidth rows="10" type="textarea" autofocus autocomplete="off" label="待转换的内容" placeholder="请输入要处理的内容" />
            <button type="button" class="nya-btn" @click="conversion">开始转换</button>
        </nya-container>
        <nya-container v-show="result" title="转换成功">
            <button type="button" class="nya-btn" @click="copy">复制</button>
            <pre>{{ result }}</pre>
        </nya-container>
    </div>
</template>
<script>

export default {
    data: () => ({
        content: '',
        result: '' //输出的结果
    }),
    methods: {
        conversion() {
            var data = this.content.split("\n");
            if(data[data.length -1] == "") {
                data.pop();
            }
            var copyContent = "";
            for (let index = 0; index < data.length; index++) {
                var pair = data[index].split(/\s+/);
                if(pair[0].length == 18 && pair[1].length == 14) {
                    copyContent += "        {";
                    copyContent += ("\n            \"card\":\"" + pair[0] + "\",");
                    copyContent += ("\n            \"pwd\":\"" + pair[1] + "\"");
                    if(index == data.length - 1)
                    {
                        copyContent += "\n        }";
                    }
                    else
                    {
                        copyContent += "\n        },\n";
                    }
                }
                else if(index != 0) {
                    alert('转换失败，异常数据：' + data[index]);
                    return;
                }
            }
            this.result = copyContent;
        },
        copy() {
            this.copyText(this.result);
        },
        copyText(value){
            const textarea = document.createElement('textarea');
            // 将该 textarea 设为 readonly 防止 iOS 下自动唤起键盘，同时将 textarea 移出可视区域
            textarea.readOnly = 'readonly';
            textarea.style.position = 'absolute';
            textarea.style.left = '-9999px';
            // 将要 copy 的值赋给 textarea 标签的 value 属性
            // 网上有些例子是赋值给innerText,这样也会赋值成功，但是识别不了\r\n的换行符，赋值给value属性就可以
            textarea.value = value;
            // 将 textarea 插入到 body 中
            document.body.appendChild(textarea);
            // 选中值并复制
            textarea.select();
            textarea.setSelectionRange(0, textarea.value.length);
            document.execCommand('Copy');
            document.body.removeChild(textarea);
        }
    }
};
</script>
