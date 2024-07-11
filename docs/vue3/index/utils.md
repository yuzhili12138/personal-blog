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