#### 获取静态文件
```
/*
 * 获取静态文件
 * @param url 文件名
 */
export const getAssetsFile = (url: string) => {
    return new URL(`../assets/images/${url}`, import.meta.url).href;
};
```

#### 批量引入图片文件
```
// eager:true 表示静态资源
// '/public/imgs'路径下
let imageList = import.meta.glob('/public/imgs/*.*',{eager:true})
console.log(Object.values(imageList).map(v=>v.default))
```



#### 获取图片内容
```
<!-- 获取所有图片 -->
export const imagesModules: Record<string, { default: string }> = import.meta.glob('../assets/images/chart/**', {
  eager: true
})

/**
 * * 获取静态图片
 * @param imageName 图片名称
 */
export const fetchImage = async (imageName:string) => {
  for (const key in imagesModules) {
    const urlSplit = key.split('/')
    if (urlSplit[urlSplit.length - 1] === imageName) {
      return imagesModules[key]?.default
    }
  }
  return ''
}


/**
 * * 获取服务器图片
 * @param imageUrl 图片路径
 */
export const fetchImageUrl = async (imageUrl:string) => {
  if (!imageUrl) return ''
  // 正则判断图片是否为 url，是则直接返回该 url
  if (/^(http|https):\/\/([\w.]+\/?)\S*/.test(imageUrl)) return imageUrl
  return `${服务器路径}${imageUrl}`
}


```



#### 获取vue组件
```
<!-- 获取所有组件 -->
const indexModules: Record<string, { default: string }> = import.meta.glob('./components/**/index.vue', {
  eager: true
})


/**
 * * 获取组件
 * @param {string} chartName 名称
 */
const fetchComponent = (chartName: string) => {
  for (const key in indexModules) {
    const urlSplit = key.split('/')
    if (urlSplit[urlSplit.length - 2] === chartName) {
      return indexModules[key]
    }
  }
}
```