在指定table高度时，若表格高度溢出的时候想有滚动条出现，必须指定el-table内置的height属性，而不是style内联属性，否则只会固定高度，不会出现滚动条

```css
:style="{maxHeight : planType == 'test' ? '100px' : '200px'}"  /* 不会出现滚动条 */

:height="tableHeight"  /* 会出现滚动条 */
```

```js
this.tableHeight = this.planType == "test" ? 300 - 50 : 200 - 50; 
```

