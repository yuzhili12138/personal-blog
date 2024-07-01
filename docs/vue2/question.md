#### 1.数据更新，视图未更新
```
## 对象
obj:{
  a:10
}
this.$set(this.obj,"a","变化的属性a")

## 数组
arr: [{ name: '宁在春', age: 21 },{ name: '北桑夏', age: 21 }]
this.$set(this.arr, 1, { name: '青冬栗', age: 23 })

```
