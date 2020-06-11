<template>
  <div class="flow-menu" ref="tool">
    <!-- <div v-for="menu  in  menuList" :key="menu.id">
      <span class="ef-node-pmenu" @click="menu.open = !menu.open">
        <i :class="{'el-icon-caret-bottom': menu.open,'el-icon-caret-right': !menu.open}"></i>
        &nbsp;{{menu.name}}
      </span>
      <ul v-show="menu.open" class="ef-node-menu-ul">
        <draggable @end="end" @start="move" v-model="menu.children" :options="draggableOptions">
          <li
            v-for="subMenu in menu.children"
            class="ef-node-menu-li"
            :key="subMenu.id"
            :type="subMenu.type"
          >
            <i :class="subMenu.ico"></i>
            {{subMenu.name}}
          </li>
        </draggable>
      </ul>
    </div>-->
    <el-menu
      default-active="0"
      class="el-menu-vertical-demo"
      @open="handleOpen"
      @close="handleClose"
    >
      <el-submenu :index="index" v-for="(menu,index)  in  menuList" :key="menu.id">
        <template slot="title">
          <i class="el-icon-s-help"></i>
          <span>{{menu.name}}</span>
        </template>
        <el-menu-item-group>
          <draggable @end="end" @start="move" v-model="menu.children" :options="draggableOptions">
            <el-menu-item
              :index="(index)-(subIndex)"
              v-for="(subMenu,subIndex) in menu.children"
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
        sort: false,
        disabled: false,
        ghostClass: 'tt',
        // 不使用H5原生的配置
        forceFallback: true,
        // 拖拽的时候样式
        fallbackClass: 'flow-node-draggable'
      },
      // 默认打开的左侧菜单的id
      defaultOpeneds: ['1', '2'],
      menuList: [
        {
          id: '1',
          type: 'group',
          name: '开始节点',
          ico: 'el-icon-video-play',
          open: true,
          // 注意type值要唯一!!!!
          children: [
            {
              id: '11',
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
              type: 'task1',
              name: '接口调用',
              ico: 'el-icon-odometer',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '13',
              type: 'task2',
              name: '接口调试',
              ico: 'el-icon-c-scale-to-original',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '14',
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
          name: '结束节点',
          ico: 'el-icon-video-pause',
          open: true,
          children: [
            {
              id: '21',
              type: 'end',
              name: '流程结束',
              ico: 'el-icon-caret-right',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '22',
              type: 'over',
              name: '数据清理',
              ico: 'el-icon-shopping-cart-full',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '23',
              type: 'over1',
              name: '数据清理',
              ico: 'el-icon-takeaway-box',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '24',
              type: 'over2',
              name: '中间件',
              ico: 'el-icon-mobile',
              state: "success",
              // 自定义覆盖样式
              style: {}
            }, {
              id: '25',
              type: 'over3',
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
    // 根据类型获取左侧菜单对象
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
    move (evt, a, b, c) {
      console.log("move", a, b, c)
      var type = evt.item.attributes.type.nodeValue
      this.nodeMenu = this.getMenuByType(type)
    },
    // 拖拽结束时触发
    end (evt, e) {
      console.log("evt", evt, e)
      console.log(this.nodeMenu)
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
