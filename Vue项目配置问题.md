在使用vue3+vite+ts学习二次封装table，import其他模块时，使用了@符号，报找不到模块的错误

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20221118002428975.png" alt="image-20221118002428975" style="zoom:50%;" />

实际上是缺少了相应的配置：

`vite.config.ts`

```typescript
import { resolve } from "path" // npm  install  path  --save 不然使用不了resolve

export default defineConfig({
  resolve: {
    // ↓路径别名，主要是这部分
    alias: {
      "@": resolve(__dirname, "./src") // npm install --save-dev @types/node  不然找不到名称dirname
    }
  }
```

`tsconfig.json`

```json
{
  "compilerOptions": {
    // 解析非相对模块名的基准目录
    "baseUrl": "./",
    // 模块名到基于 baseUrl 的路径映射的列表。
    "paths": {
      "@": ["src"],
      "@/*": ["src/*"]
   }
}
```

