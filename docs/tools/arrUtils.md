### 1.splice() slice  一般用于删除元素
```
1.splice这种方法会改变原始数组
splice(index,howmany,item1,.....,itemX)
var fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.splice(2,1,"Lemon","Kiwi");

2.slice 返回一个索引和另一个索引之间的数据(不改变原数组),slice(start,end)有两个参数(start必需,end选填),都是索引,返回值不包括end
var heroes=["李白",'蔡文姬','韩信','赵云','甄姬','阿珂','貂蝉','妲己'];
console.log(heroes.slice(1,4))//  [ "蔡文姬", "韩信", "赵云" ]开始索引为1 结束索引为4(不包括4)
```
### ！常用方法
### 1.找出符合数组下标的某一项并删除
```
 arr.splice(arr.findIndex((item) => item.id === id),1)
```


### 2.生成数组
```
序列
Array.from({length: 12}, (v, i)=> i+1)

//数组对象
Array.from({length:10},()=>{
    return {
        name:'1',
        age:'10'
   }
})

```
### 3.数组去重
```
// 正常去重
arr=['app','app2','app']
const arr1 = Array.from(new Set(arr));  
//数组对象去重
function uniqueArray(arr) {
  return Array.from(new Set(arr));
}

const arr = [{ a: 1 }, { a: 1 }];
console.log("set 不同引用：", uniqueArray(arr)); // [{ a: 1 }, { a: 1 }]

const obj1 = { a: 1 };
const arr2 = [obj1, obj1];
console.log("set 同一引用：", uniqueArray(arr2)); // [{ a: 1 }]

如果我们想认为两个对象里面的 a 属性的值相同就认为是同一数据的话，可以使用 filter

function uniqueArray(arr = [], key) {
	const keyValues = new Set();
	return arr.filter((item) => {
		if (!keyValues.has(item[key]) {
			keyValues.add(item[key]);
			return true;
		}
		return false;
	})
}
const arr = [{ a: 1 }, { a: 1 }, { a: 2 }];
console.log("filter 去重：", uniqueArray(arr, "a")); // filter 去重： [ { a: 1 }, { a: 2 } ]
```
### 3.数组交集
 ```
 const arr1 = [0, 1, 2];
const arr2 = [3, 2, 0];
function intersectSet(arr1, arr2) {
  return arr1.filter((item) => arr2.includes(item));
}
const values = intersectSet(arr1, arr2);
// 引用类型
function intersect(arr1, arr2, key) {
  const map = new Map();
  arr1.forEach((val) => map.set(val[key]));
  return arr2.filter((val) => map.has(val[key]));
}
const arr1 = [{ p: 0 }, { p: 1 }, { p: 2 }];
const arr2 = [{ p: 3 }, { p: 2 }, { p: 1 }];
const result = intersect(arr1, arr2, "p");
console.log("result:", result); // result: [ { p: 2 }, { p: 1 } ]

 ```