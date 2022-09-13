# Easy editor

## 项目介绍

`Easy Editor` 是基于 VUE+ElementUI+JsPlumb 的拓扑编辑器，通过该编辑器你可以实现拓扑图绘制,拓扑图的修改等功能

**节点拖拽功能实现流程**

通过 vuedraggable 插件来实现左侧菜单的拖拽，拖拽结束后回调 end 方法,根据鼠标所在的位置添加一个节点信息，使用 jsplumb 来管理该节点，设置该节点可拖拽、连线 并注册相关的事件。

## 效果图

![https://gitee.com/save_money/Easy-Editor/raw/master/src/assets/images/home.png](https://gitee.com/save_money/Easy-Editor/raw/master/src/assets/images/home.png)

## 功能

- 支持拖拽添加节点
- 鼠标移入到节点中显示 编辑、删除 操作
- 鼠标单击线可进行设置条件,双击可进行连线删除
- 支持预设的流程图案例切换展示
- 支持画布拖拽,控制画布网格背景
- 支持放大缩小
- 支持一键生成流程图图片
- 支持数据文件导入(目前只支持yaml文件和json文件)
- 支持显示节点内部详情
- 支持节点间回环连接,但不支持节点线连接自己

## 操作说明

- 左侧菜单子节点可以拖拽，将其拖拽到右侧画板中松开鼠标即可添加一个节点
- 鼠标点击画板中的节点上，画布右侧出现可编辑的节点详情表单,上方工具栏删除按钮功能开启，点击对应的图标可进行删除、编辑操作
- 节点分为 3 部分，节点图标在左侧，文字在中间，状态图标在右侧，鼠标从节点图标上可以拖拽出连线，可以连接其他节点，拖动图标意外位置可以改变节点在页面的位置

## 使用说明

- 【自定义节点】:左侧的节点名称以及图标可以自定义，在 node_menu.vue 页面 menuList 变量中
- 【节点添加】:左侧的菜单栏展开时，子节点可以拖拽到右侧画布中即可添加
- 【节点修改】:鼠标移动到画布中节点上，画布右侧显示用户【修改】节点的表单
- 【连线添加】:将鼠标从画布节点的左侧图标上拖动时可以拖拽出连线，然后将鼠标移动到其他节点上即可连接
- 【连线删除】:双击画布中节点之间的连线，可以进行连线删除

## 流程图数据格式

```json
{
  "name": "流程C",
  "nodeList": [
    {
      "id": "nodeA",
      "name": "流程B-节点A",
      "type": "task",
      "left": "18px",
      "top": "223px",
      "ico": "el-icon-user-solid",
      "state": "success"
    },
    {
      "id": "nodeB",
      "type": "task",
      "name": "流程B-节点B",
      "left": "351px",
      "top": "96px",
      "ico": "el-icon-goods",
      "state": "error"
    },
    {
      "id": "nodeC",
      "name": "流程B-节点C",
      "type": "task",
      "left": "354px",
      "top": "351px",
      "ico": "el-icon-present",
      "state": "warning"
    },
    {
      "id": "nodeD",
      "name": "流程B-节点D",
      "type": "task",
      "left": "723px",
      "top": "215px",
      "ico": "el-icon-present",
      "state": "running"
    }
  ],
  "lineList": [
    {
    "from": "nodeA",
    "to": "nodeB",
    "label": "条件AB",
    "connector": "Bezier",
    "paintStyle": { "strokeWidth": 2, "stroke": "#1879FF" }
    },
    {
      "from": "nodeB",
      "to": "nodeC",
      "label": "条件B"
      "connector": "Bezier",
      "paintStyle": { "strokeWidth": 2, "stroke": "#1879FF" }
    }
  ]
}
```

### data 参数说明

| 参数 |    描述    |
| :--: | :--------: |
| name | 流程图名称 |

### nodeList 参数说明

| 参数  |         描述          |              可选值              |
| :---: | :-------------------: | :------------------------------: |
|  id   |    标识唯一的节点     |                                  |
| name  |       节点名称        |                                  |
| type  |       节点类型        |                                  |
| left  | 节点在页面上的 X 坐标 |                                  |
|  top  | 节点在页面上的 Y 坐标 |                                  |
|  ico  |    节点显示的图标     |        element上的icon名         |
| state |       节点状态        | success、error、warning、running |

### lineList 参数说明

|    参数    |        描述         |                            可选值                            |
| :--------: | :-----------------: | :----------------------------------------------------------: |
|    from    | 连线的起始节点的 ID |                                                              |
|     to     |  连线的终点节点 ID  |                                                              |
|   label    |      连线说明       |                                                              |
| connector  |      连线风格       |  StateMachine、Flowchart，Bezier、Straight(默认为Flowchart)  |
| paintStyle |      连线样式       | 如:{ "strokeWidth": 2, "stroke": "#1879FF" } strokeWidth为连线宽度,stroke为连线颜色 |

## **自定义节点列表数据格式**

自定义的节点列表数据写在node_menu.vue文件中的menuList变量中

```js
 menuList: [
        {
          id: "1",
          type: "group",
          name: "接口节点",
          ico: "el-icon-video-play",
          children: [
            {
              id: "11",
              level: "1-1",
              type: "task1",
              name: "数据接入",
              ico: "el-icon-time",
              state: "success",
              // 自定义覆盖样式
              style: {
                "backgroundColor": "red"
              }
            }, {
              id: "12",
              level: "1-2",
              type: "task2",
              name: "接口调用",
              ico: "el-icon-odometer",
              state: "success",
              // 自定义覆盖样式
              style: {}
            }
          ]
        },
        {
          id: "2",
          type: "group",
          name: "工具节点",
          ico: "el-icon-video-pause",
          children: [
            {
              id: "21",
              level: "2-1",
              type: "end",
              name: "流程结束",
              ico: "el-icon-caret-right",
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: "22",
              level: "2-2",
              type: "over",
              name: "数据重置",
              ico: "el-icon-shopping-cart-full",
              state: "success",
              // 自定义覆盖样式
              style: {}
            }
          ]
        }
      ]
```

### menuList参数说明

|   参数   |         描述         |                 可选值                 |
| :------: | :------------------: | :------------------------------------: |
|    id    |    节点分组的id名    |                                        |
|   type   |         类别         |                 group                  |
|   name   |    节点分组的名称    |                                        |
|   ico    |    节点分组的图标    | element-ui中的icon名或iconfont上的名称 |
| children | 节点分组下的所有节点 |                                        |

### children参数说明

| 参数  |      描述      |                         可选值                         |
| :---: | :------------: | :----------------------------------------------------: |
|  id   | 节点分组的id名 |                                                        |
| level |    节点层级    |                                                        |
| type  |    节点类别    | 注意:这里的type值需要是唯一的,否则无法正常新增节点元素 |
| name  | 节点分组的图标 |         element-ui中的icon名或iconfont上的名称         |
| state |    节点状态    |            success、error、warning、running            |
| style | 自定义覆盖样式 |                                                        |

## 使用方式:

#### 一、以独立项目使用

```bash
# 下载工程
git clone  https://gitee.com/save_money/Easy-Editor.git

# 安装依赖包
npm install

# 启动
npm run dev

# 访问地址
 http://localhost:8080
```

##### 

## Gitee

<https://gitee.com/save_money/Easy-Editor>



## 参考资料

| 名称            | 地址                                                         | 说明         |
| --------------- | ------------------------------------------------------------ | ------------ |
| jsplumb         | <http://jsplumb.github.io/jsplumb/connections.html#sourcefilter> | 官网API文档  |
| vue-codemirror  | <https://github.surmon.me/vue-codemirror/>                   | 代码编辑器   |
| vuedraggable    | <<https://sortablejs.github.io/Vue.Draggable/#/simple>>      | 节点拖拽     |
| vue-json-viewer | <https://github.com/chenfengjw163/vue-json-viewer>           | json数据查看 |
| domtoimage      | <https://github.com/tsayen/dom-to-image#readme>              | web截图工具  |
| js-yaml         | <https://github.com/nodeca/js-yaml>                          | yaml数据转换 |
