### 1000万行表格数据实现

1. 虚拟列表（IntersectionObserver）
2. canvas实现表格
3. 可视化绘制区域算法优化
4. canvas + WebAssembly（WebAssembly 是一种新的编码方式,可以在现代的 Web 浏览器中运行——它是一种低级的类汇编语言,具有紧凑的二进制格式,可以接近原生的性能运行,并为诸如 C/C++、C# 和 Rust 等语言提供编译目标,以便它们可以在 Web 上运行。它也被设计为可以与 JavaScript 共存,允许两者一起工作。）
5. 目前有轮子（e-virt-table）

### js超过Number最大值的数怎么处理（Number.MAX_VALUE）

1. BigInt
2. decimal.js

### 如何解决页面请求接口大规模并发问题

**滑动窗口** 算法,专门用来控制流量（真实问题）

1. 请求队列
2. 防抖/节流
3. 分页加载

### 请说说大文件上传

专业术语（断点续传,断开重连,切片上传）

1. 前端切片 chunk 1024M（1048576K）,500K,const size=1048576/500
2. 将切片传给后端,切片名一般用hash加顺序下标
3. 后端组合切片

给面试官加料

- 前端切片：主进程卡顿,web-worker 多线程切片
- 切完后,将blob,存储到IndexedDB,下次用户进来之后,嗅探一下是否存在未完成上传的切换,有就尝试继续上传
- websocket,实时通知,和请求序列的控制 wss

### 在前端怎么实现截图

1. canvas
2. puppeteer(无头浏览器)
3. html2canvas

### 移动端适配问题如何解决

1. 媒体查询 （meta scale）
2. postcss  750比例
3. rem 根据html的font-size决定

### 如何修改第三方 npm 包

1. 稳定库,node_modules,直接修改
2. patch 方案  给npm包打补丁,不用自己打补丁  patch-package库
3. fork package,自己来维护

### 使用同一个链接,如何实现PC打开时web应用,手机打开是一个h5应用

识别端

```js
js识别
navigator.userAgent
```

### 当QPS 达到峰值时,该如何处理

1. 请求限流（一般不是前端处理,后端）
2. 请求合并（防抖,节流）
3. 请求缓存（请求参数,请求方法在单位时间一致,直接命中缓存 可参考alova）
4. 任务队列 （滑动窗口） （视频转码之类的,nojsjs队列可以使用bull）

### 水印（明水印,暗水印）

1. 图片水印
2. svg水印
3. 暗水印,将信息写入到文件二进制代码里面去,一般服务端实现

### web应用中如何对静态资源加载失败的场景做降级处理

1. 图片
    - 占位图,alt描述图片
    - 重试机制（404,无权限）
    - 上报

     ```js
    <img src="image.jpg" alt="example Image" onerror="handleImageErroe(this)"></img>
    ```

2. css文件  
    - 关键性样式,通过内联
    - 备用样式
    - 上报

    ```js
    <link href="styles.css" onerror="handleCssError"></link>
    function handleCssError() {
       const fallbackCss=document.createElement('link')
       fallbackCss.ref="stylesheet"
       fallbackCss.href="fallback-style.css"
       document.head.appenChild(fallbackCss)
    }
    ```

3. js文件
   - 内联脚本
   - 备用脚本处理
   - 上报

    ```js
    script onerror
    ```

4. CDN
   - 本地备份,如果cdn出错了,就是用本地备份
   - 动态切换,切到另一个有用的cdn备份
5. 字体文件
    - 使用降级字体,apple,微软雅黑
    - 使用webfont处理字体问题
6. 服务端渲染失败(ssr)
    - 降级csr

### 怎样设计一个全站请求耗时统计工具

包括首屏加载时间（FP/FCP）

1. 监控请求耗时：HTTP、中间件、axios
2. 前端监控：监控真个请求，记录耗时数据
3. 后端监控：后端记录
4. 数据汇总：数据清洗加工、数据可视化、可视化图表

```js
前端就是通过api接口将请求时间返回

后端先存表，然后写一个定时任务

```

### 请说说你对函数式编程思想的理解

函数式编程

###### 基本概念

1.完全用函数方式解决问题
2.纯函数，没有任何副作用，想通输入，得到相同输出
3.不可变性
4.高阶函数 函数柯里化
5.函数组合，类似于面向对象的继承

优点
1.可测试性，更好写单元测试
2.可维护性
3.并发
4.简洁
