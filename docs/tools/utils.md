### 验证手机号
```
/*
 * 验证手机号是否合格
 * true--说明合格
 */
export function isPhone(phoneStr) {
  let myreg = /^[1][3,4,5,7,8,9][0-9]{9}$/;
  if (!myreg.test(phoneStr)) {
    return false;
  } else {
    return true;
  }
}
```
### 验证身份证号是否合格
```
/*
 * 验证身份证号是否合格
 * true--说明合格
 */
export function isIdCard(idCardStr) {
  let idcardReg = /^[1-9]\d{7}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}$|^[1-9]\d{5}[1-9]\d{3}((0\d)|(1[0-2]))(([0|1|2]\d)|3[0-1])\d{3}([0-9]|X)$/;
  if (idcardReg.test(idCardStr)) {
    return true;
  } else {
    return false;
  }
}
```

### 验证车牌号是否合格
```
/*
 * 验证车牌号是否合格
 * true--说明合格
 */
export function isVehicleNumber(vehicleNumber) {
  let xreg = /^[京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼使领A-Z]{1}[A-Z]{1}(([0-9]{5}[DF]$)|([DF][A-HJ-NP-Z0-9][0-9]{4}$))/;
  let creg = /^[京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼使领A-Z]{1}[A-Z]{1}[A-HJ-NP-Z0-9]{4}[A-HJ-NP-Z0-9挂学警港澳]{1}$/;
  if (vehicleNumber.length == 7) {
    return creg.test(vehicleNumber);
  } else if (vehicleNumber.length == 8) {
    return xreg.test(vehicleNumber);
  } else {
    return false;
  }
}
```

### 验证字符串是否为空（也不能为纯空格）
```
/*
 * 验证字符串是否为空
 * rue--说明为空， false--说明不为空
 */
export function isEmptyString(string) {
  if (
    string == undefined ||
    typeof string == 'undefined' ||
    !string ||
    string == null ||
    string == '' ||
    /^\s+$/gi.test(string)
  ) {
    return true;
  } else {
    return false;
  }
}
```


### 生日转为年龄（精确到月份）
```
/*
 * 生日转为年龄（精确到月份）
 */
export function birsdayToAge(birsday) {
  let aDate = new Date();
  let thisYear = aDate.getFullYear();
  let bDate = new Date(birsday);
  let brith = bDate.getFullYear();
  let age = thisYear - brith;
  if (aDate.getMonth() == bDate.getMonth()) {
    if (aDate.getDate() < bDate.getDate()) {
      age = age - 1;
    }
  } else {
    if (aDate.getMonth() < bDate.getMonth()) {
      age = age - 1;
    }
  }
  return age;
}
```


### 判断数据类型
```
 * 判断数据类型
 * @param {any} val - 基本类型数据或者引用类型数据
 * @return {string} - 可能返回的结果有，均为小写字符串
 * number、boolean、string、null、undefined、array、object、function等
 */
export function getType(val){
  //判断数据是 null 和 undefined 的情况
  if (val == null) {
    return val + "";
  }
  return typeof(val) === "object" ?
      Object.prototype.toString.call(val).slice(8, -1).toLowerCase() :
      typeof(val);
}

```


### 验证是否为数字
```
/*
 * 验证是否为数字
 */
export function isNumber(n) {
  return !isNaN(parseFloat(n)) && isFinite(n);
}
```


### 是否为数组
```
/*
 * 是否为数组
 */
export function isArray(obj) {
  return Object.prototype.toString.call(obj) === '[object Array]';
}
```

### 是否空数组
```
/*
 * 是否空数组
 */
export function isArrayEmpty(val) {
  if (val && val instanceof Array && val.length > 0) {
    return false;
  } else {
    return true;
  }
}
```

### 获取url参数字符串
```
/*
 * 获取url参数字符串
 * 没有返回null
 */
export function getQueryString(name) {
  let reg = new RegExp('(^|&)' + name + '=([^&]*)(&|$)', 'i');
  let r = window.location.search.substr(1).match(reg);
  if (r != null) {
    return unescape(r[2]);
  }
  return null;
}
```


### 递归深拷贝
```
/*
 * 递归深拷贝
 */
export function deepCopy(obj) {
  let result = Array.isArray(obj) ? [] : {};
  for (let key in obj) {
    if (obj.hasOwnProperty(key)) {
      if (typeof obj[key] === 'object' && obj[key] !== null) {
        result[key] = deepCopy(obj[key]);
      } else {
        result[key] = obj[key];
      }
    }
  }
  return result;
}
```


### 函数防抖，用于将多次执行变为最后一次执行
```
/**
 * @desc  函数防抖，用于将多次执行变为最后一次执行
 * @param {function} func - 需要使用函数防抖的被执行的函数。必传
 * @param {Number} wait - 多少毫秒之内触发，只执行第一次，默认1000ms。可以不传
 */
export function debounce(fn, delay) {
  delay = delay || 1000; //默认1s后执行
  let timer = null;
  return function () {
    let context = this;
    let arg = arguments;
    if (timer) {
      clearTimeout(timer);
    }
    timer = setTimeout(() => {
      fn.apply(context, arg);
    }, delay);
  };
}
```

### 节流函数, 用于将多次执行变为每隔一段时间执行
```
/**
 * 节流函数, 用于将多次执行变为每隔一段时间执行
 * @param fn 事件触发的操作
 * @param delay 间隔多少毫秒需要触发一次事件
 */
export function throttle2(fn, delay) {
  let timer = null;
  return function () {
    let context = this;
    let args = arguments;
    if (!timer) {
      timer = setTimeout(function () {
        fn.apply(context, args);
        clearTimeout(timer);
      }, delay);
    }
  };
}
```

### 数字四舍五入（保留n位小数）
```
  export function round (number, n) {
    n = n ? parseInt(n) : 0
    if (n <= 0) return Math.round(number)
    number = Math.round(number * Math.pow(10, n)) / Math.pow(10, n)
    return number
}
```


### 数字金额转文字
```
/**
 *数字金额转文字  函数
 *
 * @export
 * @param {*} money
 * @returns
 */
export function convertCurrency(money) {
  //汉字的数字
  var cnNums = new Array('零', '壹', '贰', '叁', '肆', '伍', '陆', '柒', '捌', '玖');
  //基本单位
  var cnIntRadice = new Array('', '拾', '佰', '仟');
  //对应整数部分扩展单位
  var cnIntUnits = new Array('', '万', '亿', '兆');
  //对应小数部分单位
  var cnDecUnits = new Array('角', '分', '毫', '厘');
  //整数金额时后面跟的字符
  var cnInteger = '整';
  //整型完以后的单位
  var cnIntLast = '元';
  //最大处理的数字
  var maxNum = 999999999999999.9999;
  //金额整数部分
  var integerNum;
  //金额小数部分
  var decimalNum;
  //输出的中文金额字符串
  var chineseStr = '';
  //分离金额后用的数组，预定义
  var parts;
  if (money == '') {
    return '';
  }
  money = parseFloat(money);
  if (money >= maxNum) {
    //超出最大处理数字
    return '';
  }
  if (money == 0) {
    chineseStr = cnNums[0] + cnIntLast + cnInteger;
    return chineseStr;
  }
  //转换为字符串
  money = money.toString();
  if (money.indexOf('.') == -1) {
    integerNum = money;
    decimalNum = '';
  } else {
    parts = money.split('.');
    integerNum = parts[0];
    decimalNum = parts[1].substr(0, 4);
  }
  //获取整型部分转换
  if (parseInt(integerNum, 10) > 0) {
    var zeroCount = 0;
    var IntLen = integerNum.length;
    for (var i = 0; i < IntLen; i++) {
      var n = integerNum.substr(i, 1);
      var p = IntLen - i - 1;
      var q = p / 4;
      var m = p % 4;
      if (n == '0') {
        zeroCount++;
      } else {
        if (zeroCount > 0) {
          chineseStr += cnNums[0];
        }
        //归零
        zeroCount = 0;
        chineseStr += cnNums[parseInt(n)] + cnIntRadice[m];
      }
      if (m == 0 && zeroCount < 4) {
        chineseStr += cnIntUnits[q];
      }
    }
    chineseStr += cnIntLast;
  }
  //小数部分
  if (decimalNum != '') {
    var decLen = decimalNum.length;
    for (var i = 0; i < decLen; i++) {
      var n = decimalNum.substr(i, 1);
      if (n != '0') {
        chineseStr += cnNums[Number(n)] + cnDecUnits[i];
      }
    }
  }
  if (chineseStr == '') {
    chineseStr += cnNums[0] + cnIntLast + cnInteger;
  } else if (decimalNum == '') {
    chineseStr += cnInteger;
  }
  return chineseStr;
}
```



### 手机号码中间4位隐藏花号（*）显示
```
/*
 *手机号码中间4位隐藏花号（*）显示
 *
 */
export function hideMobile(mobile) {
  return mobile.replace(/^(\d{3})\d{4}(\d{4})$/, "$1****$2")
}
```

### 字母大小写切换
```
 /**
         * 字母大小写切换
         * @param str 要处理的字符串
         * @param type 1:首字母大写 2：首页母小写 3：大小写转换 4：全部大写 5：全部小写
         */
export function strChangeCase(str, type) {
        function ToggleCase(str) {
            var itemText = ""
            str.split("").forEach(
                function (item) {
                    if (/^([a-z]+)/.test(item)) {
                        itemText += item.toUpperCase();
                    } else if (/^([A-Z]+)/.test(item)) {
                        itemText += item.toLowerCase();
                    } else {
                        itemText += item;
                    }
                });
            return itemText;
        }

        switch (type) {
            case 1:
                return str.replace(/^(\w)(\w+)/, function (v, v1, v2) {
                    return v1.toUpperCase() + v2.toLowerCase();
                });
            case 2:
                return str.replace(/^(\w)(\w+)/, function (v, v1, v2) {
                    return v1.toLowerCase() + v2.toUpperCase();
                });
            case 3:
                return ToggleCase(str);
            case 4:
                return str.toUpperCase();
            case 5:
                return str.toLowerCase();
            default:
                return str;
        }
    }
```

### 去除字符串空格
```
        /**
         * 去除字符串空格
         * @param str 要处理的字符串
         * @param type 1：所有空格 2：前后空格 3：前空格 4：后空格
         */
export function strTrim(str, type) {
        switch (type) {
            case 1:
                return str.replace(/\s+/g, "");
            case 2:
                return str.replace(/(^\s*)|(\s*$)/g, "");
            case 3:
                return str.replace(/(^\s*)/g, "");
            case 4:
                return str.replace(/(\s*$)/g, "");
            default:
                return str;
        }
    }
```


### 返回指定某个区间的一个数字（可以不传参，不传就返回0-255之间的数；可以只传一个，返回0到传入的这个数字之间的某个值）
```
    /**
         * 返回指定某个区间的一个数字（可以不传参，不传就返回0-255之间的数；可以只传一个，返回0到传入的这个数字之间的某个值）
         * @param n1 开始区间 例：5
         * @param n2 结束区间 例：10
         * @returns 返回这个区间的某个随机值
         */
    export const randomNumber(n1, n2) {
        if (arguments.length === 2) {
            return Math.round(n1 + Math.random() * (n2 - n1));
        } else if (arguments.length === 1) {
            return Math.round(Math.random() * n1)
        } else {
            return Math.round(Math.random() * 255)
        }
    }
```

### 比较两个千分位格式化字符串的数字或百分比字符串数字
```
   /**
 * 比较两个千分位格式化字符串的数字或百分比字符串数字
 * 
 * @example 
 *  NumberCompare('101,11', '200,11', '<')
 *  NumberCompare('40%', '50.0%', '>=')
 * 
 * @param {*} arg1 数字1
 * @param {*} arg2 数字2
 * @param {*} sign 符号
 * @returns 
 */
function NumberCompare (arg1, arg2, sign = '') {
  try {
    let num1 = String(arg1).trim().replace(/[,%]/g, '')
    num1 = Number(num1)
    num1 = !isNaN(num1) ? num1 : 0

    let num2 = String(arg2).trim().replace(/[,%]/g, '')
    num2 = Number(num2)
    num2 = !isNaN(num2) ? num2 : 0

    let result = false

    switch (sign) {
      case '>':
        result = Boolean(num1 > num2)
        break
      case '>=':
        result = Boolean(num1 >= num2)
        break
      case '<':
        result = Boolean(num1 < num2)
        break
      case '<=':
        result = Boolean(num1 <= num2)
        break
      default:
        result = Boolean(num1 == num2)
        break
    }

    // console.log("NC: ", arg1, arg2, sign, num1, num2, result)
    return result

  } catch(e) {
    console.error(e)
    return false
  }
}
```
### 获取 JS 数据类型 返回大写字母开头
```
/**
 * 获取 JS 数据类型 返回大写字母开头
 * @param data
 * @returns {string} Function | Array | String | Number | Undefined | Null...
 */
export const getType = (data: any) => {
  return Object.prototype.toString.call(data).slice(8, -1)
}

```

### 获取 UUID
```
/**
 * 获取一串不会重复的字符（UUID）
 * @returns uuid {string}
 */
export const getUuid = () => {
  return Number(Math.random().toString().substr(2, 5) + Date.now()).toString(36)
}
```

### 实用 正则
```
/**
 * 1. 手机号
 * 2. 邮箱后缀
 * 3. 邮箱
 * 4. 文本
 * 5. 数字
 */
const regexp = {
  chinaMobilePhone:
    /^(\+)?(0|86|17951)?(13[0-9]||14[57]||15[012356789]||16[6]||17[678]||18[0-9]||19[89])[0-9]{8}$/,
  mailSuffix: /@[0-9a-z]+\.[0-9a-z]+$/,
  email: /^[a-zA-Z0-9.!#$%&’*+/=?^_`{|}~-]+@[a-zA-Z0-9-]+(?:\.[a-zA-Z0-9-]+)*$/,
  text: /^[0-9a-zA-Z\u4e00-\u9fa5\s]*$/,
  number: /^[0-9]*$/,
}
export default regexp
```