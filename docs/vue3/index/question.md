### vue3+vite打包时JS内存溢出的问题
```
//package.json
"scripts": {
    "build": "vite build --max_old_space_size=8192"
  },
```