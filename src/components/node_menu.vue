<template>
  <div class="flow-menu" ref="tool">
    <!-- 使用原生html结构 -->
    <!-- <div v-for="menu  in  menuList" :key="menu.id">
      <span class="ee-node-pmenu" @click="menu.open = !menu.open">
        <i :class="{'el-icon-caret-bottom': menu.open,'el-icon-caret-right': !menu.open}"></i>
        &nbsp;{{menu.name}}
      </span>
      <ul v-show="menu.open" class="ee-node-menu-ul">
        <draggable @end="end" @start="move" v-model="menu.children" :options="draggableOptions">
          <li
            v-for="subMenu in menu.children"
            class="ee-node-menu-li"
            :key="subMenu.id"
            :type="subMenu.type"
          >
            <i :class="subMenu.ico"></i>
            {{subMenu.name}}
          </li>
        </draggable>
      </ul>
    </div>-->
    <!-- 使用element导航菜单 -->
    <el-menu
      class="el-menu-vertical-demo"
      @open="handleOpen"
      @close="handleClose"
      :default-openeds="['1','2']"
    >
      <el-submenu :index="menu.id" v-for="menu  in  menuList" :key="menu.id">
        <template slot="title">
          <i class="el-icon-s-help"></i>
          <span>{{menu.name}}</span>
        </template>
        <el-menu-item-group>
          <draggable @end="end" @start="move" v-model="menu.children" :options="draggableOptions">
            <el-menu-item
              :index="subMenu.level"
              v-for="subMenu in menu.children"
              :key="subMenu.id"
              :type="subMenu.type"
            >
              <i :class="subMenu.ico" style="color:#1879ff"></i>
              {{subMenu.name}}
            </el-menu-item>
          </draggable>
        </el-menu-item-group>
      </el-submenu>
    </el-menu>
  </div>
</template>
<script>
import draggable from 'vuedraggable'

var mousePosition = {
  left: -1,
  top: -1
}

export default {
  data () {
    return {
      activeNames: '1',
      // draggable配置参数参考 https://www.cnblogs.com/weixin186/p/10108679.html
      draggableOptions: {
        preventOnFilter: false,
        //定义是否列表单元是否可以在列表容器内进行拖拽排序
        sort: false,
        disabled: false,
        // 当拖动列表单元时会生成一个副本作为影子单元来模拟被拖动单元排序的情况，此配置项就是来给这个影子单元添加一个class，我们可以通过这种方式来给影子元素进行编辑样式
        ghostClass: 'tt',
        chosenClass: 'choosen',//当选中列表单元时会给该单元增加一个class
        // 不使用H5原生的配置
        forceFallback: true,
        // 拖放过程中鼠标附着单元的样式
        fallbackClass: 'flow-node-draggable'
      },
      // 默认打开的左侧菜单的id
      defaultOpeneds: ['1', '2'],
      menuList: [
        {
          id: '1',
          type: 'group',
          name: '接口节点',
          ico: 'el-icon-video-play',
          // open: true,
          // 注意type值要唯一!!!!
          children: [
            {
              id: '11',
              level: "1-1",
              type: 'timer',
              name: '数据接入',
              ico: 'el-icon-time',
              state: "success",
              // 自定义覆盖样式
              style: {
                'backgroundColor': 'red'
              }
            }, {
              id: '12',
              level: "1-2",
              type: 'task1',
              name: '接口调用',
              ico: 'el-icon-odometer',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '13',
              level: "1-3",
              type: 'task2',
              name: '接口调试',
              ico: 'el-icon-c-scale-to-original',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '14',
              level: "1-4",
              type: 'timer2',
              name: '接口配置',
              ico: 'el-icon-files',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }
          ]
        },
        {
          id: '2',
          type: 'group',
          name: '工具节点',
          ico: 'el-icon-video-pause',
          // open: true,
          children: [
            {
              id: '21',
              level: "2-1",
              type: 'end',
              name: '流程结束',
              ico: 'el-icon-caret-right',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '22',
              level: "2-2",
              type: 'over',
              name: '数据重置',
              ico: 'el-icon-shopping-cart-full',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '23',
              level: "2-3",
              type: 'clean',
              name: '数据清理',
              ico: 'el-icon-takeaway-box',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '24',
              level: "2-4",
              type: 'middle',
              name: '中间件',
              ico: 'el-icon-mobile',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '25',
              level: "2-5",
              type: 'system',
              name: '系统配置',
              ico: 'el-icon-coordinate',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }
          ]
        }
      ],
      nodeMenu: {}
    }
  },
  components: {
    draggable
  },
  created () {
    /**
     * 以下是为了解决在火狐浏览器上推拽时弹出tab页到搜索问题
     * @param event
     */
    if (this.isFirefox()) {
      document.body.ondrop = function (event) {
        // 解决火狐浏览器无法获取鼠标拖拽结束的坐标问题
        mousePosition.left = event.layerX
        mousePosition.top = event.clientY - 50
        event.preventDefault();
        event.stopPropagation();
      }
    }
  },
  methods: {
    handleOpen (key, keyPath) {
      console.log(key, keyPath);
    },
    handleClose (key, keyPath) {
      console.log(key, keyPath);
    },
    // 根据类型获取左侧菜单被拖动的节点
    getMenuByType (type) {
      for (let i = 0; i < this.menuList.length; i++) {
        let children = this.menuList[i].children;
        for (let j = 0; j < children.length; j++) {
          if (children[j].type === type) {
            return children[j]
          }
        }
      }
    },
    // 拖拽开始时触发
    move (evt) {
      var type = evt.item.attributes.type.nodeValue
      // console.log(type)
      this.nodeMenu = this.getMenuByType(type)
      console.log(this.nodeMenu)
    },
    // 拖拽结束时触发
    end (evt) {
      this.$emit('addNode', evt, this.nodeMenu, mousePosition)
    },
    // 是否是火狐浏览器
    isFirefox () {
      var userAgent = navigator.userAgent
      if (userAgent.indexOf("Firefox") > -1) {
        return true
      }
      return false
    }
  }
}
</script>
<style lang="less">
.choosen {
  border: 1px solid #e0e3e7;
  border-radius: 5px;
  background-color: #fff;
}
</style>