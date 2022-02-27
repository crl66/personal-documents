### $ref(告别.value)和自动引入api

```js
// vite.config.js
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import AutoImport from 'unplugin-auto-import/vite'
export default defineConfig({ 
  plugins: [vue({ reactivityTransform: true }),
    AutoImport({
      // dts: 'src/auto-imports.d.ts', // 可以自定义文件生成的位置，默认是根目录下
      imports: ['vue']
    })
  ] 
})

```



### 组件按需引入(组件,ui(Element-ui)库,vue hooks等)

```
https://juejin.cn/post/7012446423367024676
```

