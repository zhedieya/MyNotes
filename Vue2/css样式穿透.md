##### vue2.x

```css
第一种写法箭头三剑客（原生css）：>>>
.类名 >>> .类名{ 样式 }

第二种（预处理器：sass、less）：/deep/
/deep/ .类名{ 样式 }

第三种（预处理器：sass、less）：::v-deep
::v-deep .类名{ 样式 }
```

##### vue3.x

```css
第一种：:deep()
:deep( .类名 ){ 样式 }

第二种：::v-deep()
::v-deep( .类名 ){ 样式 }
```

##### 插槽选择器

```css
:slotted( .类名 )
```

##### 全局选择器

```css
:global( .类名 )
```