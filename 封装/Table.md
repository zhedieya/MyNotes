在公司经常接触到后台管理系统的开发，大部分业务都会涉及表格table，业务逻辑也基本一样，每次遇到table的需求又得重新写一遍，代码重复又费时费力，因此，根据业务需求将常用的组件二次封装是最佳解决方法，有如下几个好处：

- 降低系统各个功能的耦合性
- 提高了功能内部的聚合性
- 前端工程化
- 代码维护难度降低
- 开发效率以及开发成本的提升

查阅资料后，我也得出了二次封装组件的一些心得，在这里分享给大家

### 目录结构

首先是目录结构



### 表格搜索区域需求分析：

> 一般来说，搜索区域的字段都是存在于表格当中的，并且每个页面的搜索、重置方法都是一样的逻辑，只是不同的查询参数而已。我们完全可以在传表格配置项 **columns** 时，直接指定某个 **column** 的 **search** 配置，就能把该项变为搜索项，然后使用 **el** 字段可以指定搜索框的类型，最后把表格的搜索方法都封装成 **Hooks** 钩子函数。页面上完全就不会存在任何搜索、重置逻辑了。

```vue
<SearchForm :search="search" :reset="reset" :searchParam="searchParam" :columns="searchColumns" v-show="isShowSearch" />
```

首先来看一下表格搜索区域的配置，在这里重要的就是columns属性，该属性配置搜索表单的默认值			

```typescript
// 扁平 columns
const flatColumns = ref<ColumnProps[]>();
flatColumns.value = flatColumnsFunc(tableColumns.value);

// 过滤需要搜索的配置项 && 处理搜索排序
const searchColumns = flatColumns.value
	.filter(item => item.search?.el)
	.sort((a, b) => (b.search?.order ?? 0) - (a.search?.order ?? 0));

// 设置搜索表单的默认值
searchColumns.forEach(column => {
	if (column.search?.defaultValue !== undefined && column.search?.defaultValue !== null) {
		searchInitParam.value[column.search.key ?? column.prop!] = column.search?.defaultValue;
	}
});
```

