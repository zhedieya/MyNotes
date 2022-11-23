在公司经常接触到后台管理系统的开发，大部分业务都会涉及表格table，业务逻辑也基本一样，每次遇到table的需求又得重新写一遍，代码重复又费时费力，因此，在项目开始前根据业务需求将常用的组件二次封装是最佳解决方法，有如下几个好处：

- 降低系统各个功能的耦合性
- 提高了功能内部的聚合性
- 前端工程化
- 代码维护难度降低
- 开发效率以及开发成本的提升

查阅资料后，我也得出了二次封装组件的一些心得，在这里分享给大家

**这是最终的效果**

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20221120185301565.png" alt="image-20221120185301565" style="zoom:50%;" />

我会对该页面的每个区域进行分析，讲述实现的步骤与方法。

### 基础介绍

#### 页面使用二次封装的Table

useComponent里是对Table的使用案例

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20221120191054772.png" alt="image-20221120191054772" style="zoom:50%;" />

由此可见，每个需要使用表格的地方都可以按需灵活配置，基础的表格只需传递表格列配置项columns，组件会根据此字段渲染搜索表单与表格列，表格数据请求的api，传递dataCallback对返回的表格数据做处理以及表格初始化的请求参数initParams便可，其余的功能可通过插槽来实现。

##### 表格列配置项columns

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20221120192228595.png" alt="image-20221120192228595" style="zoom:50%;" />

table组件会接收到作为props传来的columns，并设置为响应式，之后便可扁平化columns并处理设置为响应式后的数据，比如若某一列的数据需要额外调用函数处理，在设置columns数据时便可**传递函数/接口**过去，在table组件里调用接口。

```js
{
  prop: "status",
  label: "用户状态",
  enum: getUserStatus, // 函数/接口
  fieldNames: { label: "userLabel", value: "userStatus" },
  search: {
    el: "tree-select",
    props: { props: { label: "userLabel" }, nodeKey: "userStatus" }
 },
```

```js
// 如果当前 enum 为后台数据需要请求数据，则调用该请求接口，并存储到 enumMap
if (typeof col.enum !== "function") return enumMap.value.set(col.prop!, col.enum);
const { data } = await col.enum();
enumMap.value.set(col.prop!, data);
```

#### 二次封装的Table组件

`Table.vue`

**查询表单**

```vue
<!-- 查询表单 card -->
<SearchForm :search="search" :reset="reset" :searchParam="searchParam" :columns="searchColumns" v-show="isShowSearch" />
```

可过滤出含有需要搜索的配置项作为props传入columns，将搜索表单的默认值作为props传入searchParam，传入表单搜索和重置功能，这样基础的查询表单需要的配置就都齐全了。

| 插槽名                   | 描述                                 |
| ------------------------ | ------------------------------------ |
| —                        | 默认插槽，支持直接写 el-table-column |
| tableHeader              | 自定义表格头部左侧区域               |
| `column.prop`            | 单元格的插槽                         |
| `column.prop` + "Header" | 表头的插槽                           |

**表格头部的操作按钮**

使用具名作用域插槽，可获取选中的表格数据，进行进一步操作

```vue
<!-- 表格头部 操作按钮 -->
<div class="table-header">
  <div class="header-button-lf">
    <!-- 具名作用域插槽，子组件数据会向一个插槽的出口上传递 attributes： -->
    <slot name="tableHeader" :selectedListIds="selectedListIds" :selectList="selectedList" :isSelected="isSelected"></slot>
  </div>
  <div class="header-button-ri" v-if="toolButton">
    <el-button :icon="Refresh" circle @click="getTableList"> </el-button>
    <el-button :icon="Printer" circle v-if="columns.length" @click="handlePrint"> </el-button>
    <el-button :icon="Operation" circle v-if="columns.length" @click="openColSetting"> </el-button>
    <el-button :icon="Search" circle v-if="searchColumns.length" @click="isShowSearch = !isShowSearch"> </el-button>
  </div>
</div>
```

**表格主体**

循环渲染列表列

除开固定的几列(如selection，index，expand等)，其余的数据列通过内循环在TableColumn里来渲染，TableColumn里通过动态组件渲染。

```vue
    <!-- 表格主体 -->
		<el-table
			ref="tableRef"
			v-bind="$attrs"
			:data="tableData"
			:border="border"
			:row-key="getRowKeys"
			@selection-change="selectionChange"
		>
			<!-- 默认插槽 -->
			<slot></slot>
			<template v-for="item in tableColumns" :key="item">
				<!-- selection || index -->
				<el-table-column
					v-bind="item"
					:align="item.align ?? 'center'"
					:reserve-selection="item.type == 'selection'"
					v-if="item.type == 'selection' || item.type == 'index'"
				>
				</el-table-column>
				<!-- expand 支持 tsx 语法 && 作用域插槽 (tsx > slot) -->
				<el-table-column v-bind="item" :align="item.align ?? 'center'" v-if="item.type == 'expand'" v-slot="scope">
					<!-- tsx语法渲染 -->
					<component :is="item.render" :row="scope.row" v-if="item.render"> </component>
					<!-- 作用域插槽渲染 -->
					<slot :name="item.type" :row="scope.row" v-else></slot>
				</el-table-column>
				<!-- other 循环递归 -->
				<!-- 目前的理解是 TableColumn实际上就是根据不同属性通过tsx缔造的table-coloumn 实际上就是不同的el-table-column包裹着底下的template -->
				<TableColumn v-if="!item.type && item.prop && item.isShow" :column="item">
					<!-- $slots表示父组件所传入插槽的对象 -->
					<template v-for="slot in Object.keys($slots)" #[slot]="scope">
						<slot :name="slot" :row="scope.row"></slot>
					</template>
				</TableColumn>
			</template>
			<!-- 无数据 -->
			<template #empty>
				<div class="table-empty">
					<img src="@/assets/images/notData.png" alt="notData" />
					<div>暂无数据</div>
				</div>
			</template>
		</el-table>
```

renderLoop返回用tsx语法渲染的表格column组件，默认渲染普通的column，也会根据插槽slot名称来判断是渲染表头还是表数据

```vue
<template>
	<!-- 动态组件 -->·
	<component :is="renderLoop(column)"></component>
</template>
```

传递进renderLoop里具体的item内容：

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20221122233456774.png" alt="image-20221122233456774" style="zoom:50%;" />

rederLoop会根据item里的内容来判断是渲染普通的表格还是通过tsx语法渲染的表头或者单元格

<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20221123223156876.png" alt="image-20221123223156876" style="zoom:50%;" />





<img src="https://raw.githubusercontent.com/zhedieya/MyPics/main/typora-img/image-20221122233248275.png" alt="image-20221122233248275" style="zoom:50%;" />





分页

```vue
	<!-- 分页组件 -->
		<Pagination
			v-if="pagination"
			:pageable="pageable"
			:handleSizeChange="handleSizeChange"
			:handleCurrentChange="handleCurrentChange"
		/>
```



#### hooks

> 🤔 **前提：首先我们在封装 ProTable 组件的时候，在不影响 el-table 原有的属性、事件、方法的前提下，然后在其基础上做二次封装，否则做得再好，也不太完美。**

> 🧐 **思路：把一个表格页面所有重复的功能 （表格多选、查询、重置、刷新、分页、数据操作二次确认、文件下载、文件上传等） 都封装成 Hooks 函数钩子或组件，然后在 Table 组件中使用这些函数钩子或组件。在页面中使用的时，只需传给 Table 当前表格数据的请求 API、表格配置项 columns 就行了，数据传输都使用 作用域插槽 或 tsx 语法从 Table 传递给父组件就能在页面上获取到了。**

**常用 Hooks 函数**

- **useTable：**

  useTable.ts里是一个函数，参数参数为:

  ```js
  /**
   * @description table 页面操作方法封装
   * @param {Function} api 获取表格数据 api 方法(必传)
   * @param {Object} initParam 获取数据初始化参数(非必传，默认为{})
   * @param {Boolean} isPageable 是否有分页(非必传，默认为true)
   * @param {Function} dataCallBack 对后台返回的数据进行处理的方法(非必传)
   * */
  ```

  函数里定义了表格的相关属性，比如表格数据、分页数据等 ; 以及相关的操作函数，比如获取表格数据、表格数据查询重置等；

  函数的返回值是：

  ```js
  return {
    ...toRefs(state), // 将响应式对象转换为普通对象返回
    getTableList,
    search,
    reset,
    handleSizeChange,
    handleCurrentChange
  };
  ```

  之后便可在Table组件里使用，解构获取hooks函数返回的内容

  ```js
  // 表格操作 Hooks
  const { tableData, pageable, searchParam, searchInitParam, getTableList, search, reset, handleSizeChange, handleCurrentChange } =
  	useTable(props.requestApi, props.initParam, props.pagination, props.dataCallback);
  ```

- **useSelection**

  ```js
  /**
   * @description 表格多选数据操作
   * @param {String} selectId 当表格可以多选时，所指定的 id
   * @param {Any} tableRef 当表格 ref
   */
  ```

  函数里定义了选中数据的相关属性，比如选中的数据、选中的数据id等，以及最重要的selection-change函数，还有获取行数据Key的函数,用来优化 Table 的渲染;在使用跨页多选时,该属性是必填的

    ```js
    return {
      isSelected,
      selectedList,
      selectedListIds,
      selectionChange,
      getRowKeys
    };
    ```

  之后便可在Table组件里使用，解构获取hooks函数返回的内容

  ```js
  // 表格多选 Hooks
  const { selectionChange, getRowKeys, selectedList, selectedListIds, isSelected } = useSelection(props.selectId);
  ```

  

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

### 表格数据操作按钮区域需求分析：

> 表格数据操作按钮基本上每个页面都会不一样，所以我们直接使用 **作用域插槽** 来完成每个页面的数据操作按钮区域，**作用域插槽** 可以将表格多选数据信息从 **Table** 的 **Hooks** 多选钩子函数中传到页面上使用。

> **scope** 数据中包含：**selectedList（当前选择的数据）、selectedListIds（当前选择的数据id）、isSelected（当前是否选中的数据）**

```html
<!-- ProTable 中 tableHeader 插槽 -->
<slot name="tableHeader" :selectList="selectedList" :selectedListIds="selectedListIds" :isSelected="isSelected"></slot>

<!-- 页面使用 -->
<template #tableHeader="scope">
    <el-button type="primary" :icon="CirclePlus" @click="openDrawer('新增')">新增用户</el-button>
    <el-button type="primary" :icon="Upload" plain @click="batchAdd">批量添加用户</el-button>
    <el-button type="primary" :icon="Download" plain @click="downloadFile">导出用户数据</el-button>
    <el-button type="danger" :icon="Delete" plain @click="batchDelete(scope.selectedListIds)" :disabled="!scope.isSelected">批量删除用户</el-button>
</template>
```

### 表格主体内容展示区域分析：

> 🍉 该区域是最重要的数据展示区域，对于使用最多的功能就是表头和单元格内容可以自定义渲染

> 💥 表头支持`headerRender`方法（避免与 **el-table-column** 上的属性重名导致报错）、作用域插槽（`column.prop + 'Header'`）两种方式自定义，单元格内容支持`render`方法和作用域插槽（`column 上的 prop 属性`）两种方式自定义。

- 使用作用域插槽：

```html
<!-- 使用作用域插槽自定义单元格内容 username -->
<template #username="scope">
    {{ scope.row.username }}
</template>

<!-- 使用作用域插槽自定义表头内容 username -->
<template #usernameHeader="scope">
    <el-button type="primary" @click="ElMessage.success('我是通过作用域插槽渲染的表头')">
        {{ scope.row.label }}
    </el-button>
</template>
```

- 使用 tsx 语法：

```html
<script setup lang="tsx">
const columns: ColumnProps[] = [
 {
    prop: "username",
    label: "用户姓名",
    // 使用 headerRender 自定义表头
    headerRender: (row) => {
      return (
        <el-button
          type="primary"
          onClick={() => {
            ElMessage.success("我是通过 tsx 语法渲染的表头");
          }}
        >
          {row.label}
        </el-button>
      );
    }
  },
  {
    prop: "status",
    label: "用户状态",
    // 使用 render 自定义表格内容
    render: (scope: { row }) => {
      return (
          <el-switch
            model-value={scope.row.status}
            active-text={scope.row.status ? "启用" : "禁用"}
            active-value={1}
            inactive-value={0}
            onClick={() => changeStatus(scope.row)}
          />
        ) 
      );
    }
  },
];
</script>
```

> 💢💢💢 **最强大的功能：如果你想使用 `el-table` 的任何属性、事件，目前通过属性透传都能支持。**
>
> **如果你还不了解属性透传，请阅读 vue 官方文档：[cn.vuejs.org/guide/compo…](https://link.juejin.cn?target=https%3A%2F%2Fcn.vuejs.org%2Fguide%2Fcomponents%2Fattrs.html)**

- **Table 组件上的绑定的所有属性和事件都会通过 `v-bind="$attrs"` 透传到 el-table 上。**
- **Table 组件内部暴露了 el-table DOM，可通过 `proTable.value.element.方法名` 调用其方法。**

```html
<template>
    <el-table
      ref="tableRef"
      v-bind="$attrs" 
    >
    </el-table>
</template>

<script setup lang="ts" name="ProTable">
import { ref } from "vue";
import { ElTable } from "element-plus";

const tableRef = ref<InstanceType<typeof ElTable>>();

defineExpose({ element: tableRef });
</script>
```

### 表格分页区域分析：

```html
<template>
  <!-- 分页组件 -->
  <el-pagination
    :current-page="pageable.pageNum"
    :page-size="pageable.pageSize"
    :page-sizes="[10, 25, 50, 100]"
    :background="true"
    layout="total, sizes, prev, pager, next, jumper"
    :total="pageable.total"
    @size-change="handleSizeChange"
    @current-change="handleCurrentChange"
  ></el-pagination>
</template>

<script setup lang="ts" name="pagination">
interface Pageable {
  pageNum: number;
  pageSize: number;
  total: number;
}

interface PaginationProps {
  pageable: Pageable;
  handleSizeChange: (size: number) => void;
  handleCurrentChange: (currentPage: number) => void;
}

defineProps<PaginationProps>();
</script>
```
