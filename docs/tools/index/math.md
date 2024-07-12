

### 将输入转换为数字，如果不是数字或字符串则返回NaN
```
/**
 * 将输入转换为数字，如果不是数字或字符串则返回NaN
 * @param {any} num - 输入值
 * @returns {number} 转换后的数字
 */
function convertToNumber(num: any): number {
  if (typeof num === "number") {
    return num;
  } else if (typeof num === "string") {
    return parseFloat(num);
  }
  return NaN;
}

```

### 保留数字小数点 四舍五入
```
/*
 * 保留数字小数点
 * num--数字
 * n--保留的小数点
 */
export const fomatFloat = (num, n = 2) => {
    var f = parseFloat(num);
    if (isNaN(f)) {
        return 0;
    }
    f = Math.round(num * Math.pow(10, n)) / Math.pow(10, n); // n 幂   
    var s = f.toString();
    var rs = s.indexOf('.');
    //判定如果是整数，增加小数点再补0
    // if (rs < 0) {
    //     rs = s.length;
    //     s += '.';
    // }
    while (s.length <= rs + n) {
        s += '0';
    }
    return s;
}  
```
### 格式化百分比（直接截取）
```
/**
 * 格式化百分比，将数字转换为百分比形式
 * @param {number|string} num - 要格式化的数字
 * @param {number} precision - 小数点后保留的位数，默认为2
 * @returns {string} 格式化后的百分比字符串
 */
export const formatPercent = (num: number | string, precision = 2): string => {
  num = convertToNumber(num);
  if (isNaN(num)) return "0%";
  return (num * 100).toFixed(precision) + "%";
};
```

### 格式化百分比（四舍五入）
```
/**
 * 格式化百分比，将数字转换为百分比形式
 * @param {number|string} num - 要格式化的数字
 * @param {number} precision - 小数点后保留的位数，默认为2
 * @returns {string} 格式化后的百分比字符串
 */
export const formatPercent = (num: number | string, precision = 2): string => {
  num = convertToNumber(num);
  if (isNaN(num)) return "0%";
  return fomatFloat(num * 100,precision) +'%'
};
```

### 添加千位分隔符到数字字符串
```
/**
 * 添加千位分隔符到数字字符串
 * @param {number|string} num - 数字或数字字符串
 * @returns {string} 添加分隔符后的字符串
 */
function formatWithThousandsSeparator(num: number | string): string {
  const n = typeof num === "number" ? num.toString() : num;
  if (isNaN(Number(n))) return "-";
  const [integer, decimal] = n.split(".");
  const reg = /\d{1,3}(?=(\d{3})+$)/g;
  const n1 = integer.replace(reg, "$&,");
  return decimal ? `${n1}.${decimal}` : n1;
}
```

### 格式化货币，自动转换为“万”或“亿”单位
```
/**
 * 格式化货币，自动转换为“万”或“亿”单位
 * @param {number|string} num - 要格式化的数字
 * @param {string} unit - 货币单位，默认为“元”
 * @returns {string} 格式化后的货币字符串
 */
const formatCurrency = (num: number | string, unit = "元"): string => {
  num = convertToNumber(num);
  if (isNaN(num)) return "-";
  let amount = "";
  if (Math.abs(num) >= 1000000000.0) {
    num /= 100000000.0;
    amount = "亿";
  } else if (Math.abs(num) >= 100000.0) {
    num /= 10000.0;
    amount = "万";
  }
  return formatWithThousandsSeparator(num.toFixed(2)) + " " + amount + unit;
};
```




