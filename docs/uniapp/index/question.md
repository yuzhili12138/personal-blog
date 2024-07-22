#### uniapp中引用uView后，使用u-input标签的type=‘number‘在小程序失效
```
<!-- 添加change事件 -->
        <u--input
              inputAlign="right"
              @change="formatTextToNumber"
              v-model="form.projectDuration"
              border="none"
              placeholder="小时"
              type="number"
            ></u--input>

<!-- 正则处理数据 -->
 formatTextToNumber(val) {
      let temp = val.replace(/[^0-9.]+/g, "");
      this.$nextTick(() => {
        this.form.projectDuration = temp;
      });
    },            

```
