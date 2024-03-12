#### 获取静态文件
```
/*
 * 验证手机号是否合格
 * true--说明合格
 */
export const getAssetsFile = (url: string) => {
    return new URL(`../assets/images/${url}`, import.meta.url).href;
};
```