# 简介
`unisoc-ui`组件包是对ElementUI进行的二次封装，因此只适用于Vue项目，其用法跟ElementUI的用法基本一致。

# 第一步：npm环境配置
由于`unisoc-ui`是发布在公司私有npm上，所以下载前需要配置本地环境至内网，终端运行下面命令：

```javascript
npm config set registry https://repo.unisoc.com/nexus/repository/npm-public/
```

# 第二步：安装unisoc-ui包
在需要用到该组件包的项目的根目录下运行下面命令：

```javascript
npm i unisoc-ui --save
```

# 第三步： 引入组件包
安装好依赖包后，在项目的`main.js`中引入`unisoc-ui`和`element-ui`并引入unisoc-ui中的**样式文件**:

```javascript
// main.js
import ElementUI from 'element-ui';
import unisocUI from 'unisoc-ui'
import 'unisoc-ui/style-default/index'

Vue.use(unisocUI);
Vue.use(ElementUI);
```

> 1. unisoc-ui的依赖中添加了@2.14.1版本的element-ui，在下载unisoc-ui的同时会一并下载到element-ui，无需单独下载。
> 2. unisoc-ui的样式文件中包含了element-ui的样式文件，所以无需额外引入element-ui的样式文件。

# 第四步：使用组件
组件包demo地址: [unisoc-ui-demo](http://10.29.60.94:8282/#/).
请查阅上面demo地址中所有组件的具体使用方法，在页面上直接粘贴组件标签，并传入相应的数据或方法即可 ==（同element-ui用法）==。

## 用table用法举例：
```javascript
// HomePage.vue
<template>
  <div class="home">
    <operate-table
      :dataSource="operateTableData"
      :columns="operateTableColumns"
      :options="operateTableOption"
      :operates="operateTableOperates"
      ></operate-table>
  </div>
</template>

<script>
// @ is an alias to /src
export default {
  name: 'Home',
  methods: {
     handleDelete(index, row) {
     	this.$message.info(
         "这里写删除逻辑 index:" + index + " row:" + JSON.stringify(row)
       );
     },
     handleEdit(index, row) {
       this.$message.info(
         "这里写弹出框编辑逻辑 index:" + index + " row:" + JSON.stringify(row)
       );
     },
     handleSwitch(index, row) {
       this.$message.info(
         "这里写switch change逻辑 index:" + index + " row:" + JSON.stringify(row)
       );
     },
  },
  data () {
    return {
      operateTableColumns: [
        {
          prop: "projectname",
          label: "项目名",
          align: "center",
          minwidth: 70,
          formatter: (row) => {
            return `<a href="http://10.29.60.125:8060/" target="_blank">${row.projectname}</a>`;
          },
        },
        {
          prop: "version",
          label: "版本号",
          align: "center",
          width: 70,
          resizable: false,
          render: (h, params) => {
            return h(
              "el-tag",
              {
                props: {
                  type: params.row.version > 2 ? "success" : "danger",
                }, // 组件的props
              },
              params.row.version
            );
          },
        },
        {
          prop: "finno",
          label: "财务编号",
          align: "center",
          width: 150,
        },
        {
          prop: "psename",
          label: "PSE",
          align: "center",
        },
      ],
      operateTableData: [
        {
          id: 1,
          projectname: "LTE终端高性能多频滤波器研发",
          version: "1",
          finno: "P20068E",
          psename: "梁聪 (Cong Liang)",
          switchValue: false,
        },
        {
          id: 2,
          projectname: "QogirN8",
          version: "2",
          finno: "P15049E",
          psename: "杨鹤 (He Yang)",
          switchValue: true,
        },
        {
          id: 3,
          projectname: "Info_内外网站改版",
          version: "3",
          finno: "NA",
          psename: "熊香玲 (Shelly Xiong)",
          switchValue: true,
        },
      ],
      operateTableOption: {
        stripe: true, // 是否为斑马纹 table
        highlightCurrentRow: false, // 是否要高亮当前行
      },
      operateTableOperates: {
        list: [
          {
            id: "1",
            type: "switch",
            show: true,
            method: (index, row) => {
              this.handleSwitch(index, row);
            },
            prop: "switchValue",
          },
          {
            id: "2",
            type: "tooltipIcon",
            tooltip: "删除",
            show: true,
            icon: "iconfont iconshanchu",
            disabled: true,
            method: (index, row) => {
              this.handleDelete(index, row);
            },
          },
          {
            id: "3",
            type: "icon",
            show: true,
            icon: "iconfont icontubiao09",
            disabled: false,
            method: (index, row) => {
              this.handleEdit(index, row);
            },
          },
        ],
        width: "120",
       }
    }
  },
}
</script>

```
## 效果：
![效果](https://img-blog.csdnimg.cn/20210317102654817.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2hlbGxlbll1YW4=,size_16,color_FFFFFF,t_70)


*问题反馈：tonghui.yuan@unisoc.com*