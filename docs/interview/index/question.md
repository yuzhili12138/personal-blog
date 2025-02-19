### 1000万行表格数据实现

1. 虚拟列表（IntersectionObserver）
2. canvas实现表格
3. 可视化绘制区域算法优化
4. canvas + WebAssembly（WebAssembly 是一种新的编码方式，可以在现代的 Web 浏览器中运行——它是一种低级的类汇编语言，具有紧凑的二进制格式，可以接近原生的性能运行，并为诸如 C/C++、C# 和 Rust 等语言提供编译目标，以便它们可以在 Web 上运行。它也被设计为可以与 JavaScript 共存，允许两者一起工作。）
5. 目前有轮子（e-virt-table）

### js超过Number最大值的数怎么处理（Number.MAX_VALUE）

1. BigInt
2. decimal.js

### 如何解决页面请求接口大规模并发问题

**滑动窗口** 算法，专门用来控制流量（真实问题）

1. 请求队列
2. 防抖/节流
3. 分页加载

### 请说说大文件上传

专业术语（断点续传，断开重连，切片上传）

1. 前端切片 chunk 1024M（1048576K），500K，const size=1048576/500
2. 将切片传给后端，切片名一般用hash加顺序下标
3. 后端组合切片

给面试官加料

- 前端切片：主进程卡顿，web-worker 多线程切片
- 切完后，将blob，存储到IndexedDB，下次用户进来之后，嗅探一下是否存在未完成上传的切换，有就尝试继续上传
- websocket，实时通知，和请求序列的控制 wss

### 在前端怎么实现截图

1. canvas
2. puppeteer(无头浏览器)
3. html2canvas

### 移动端适配问题如何解决

1. 媒体查询 （meta scale）
2. postcss  750比例
3. rem 根据html的font-size决定

### 如何修改第三方 npm 包

1. 稳定库，node_modules，直接修改
2. patch 方案  给npm包打补丁，不用自己打补丁  patch-package库
3. fork package，自己来维护

### 使用同一个链接，如何实现PC打开时web应用，手机打开是一个h5应用

识别端

```
js识别
navigator.userAgent
```

### 当QPS 达到峰值时，该如何处理

1. 请求限流（一般不是前端处理，后端）
2. 请求合并（防抖，节流）
3. 请求缓存（请求参数，请求方法在单位时间一致，直接命中缓存 可参考alova）
4. 任务队列 （滑动窗口） （视频转码之类的，nojsjs队列可以使用bull）

### 水印（明水印，暗水印）

1. 图片水印
2. svg水印
3. 暗水印，将信息写入到文件二进制代码里面去,一般服务端实现
