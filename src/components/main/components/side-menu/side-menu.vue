<template>
  <div class="side-menu-wrapper">
    <slot></slot>
    <Menu ref="menu" v-show="!collapsed" :active-name="activeName" :open-names="openedNames" :accordion="accordion" :theme="theme" width="auto" @on-select="handleSelect">
      <template v-for="item in menuList">
        <!-- 只有一个子菜单 -->
        <template v-if="item.children && item.children.length === 1">
          <!-- showChildren 在mixin中-->
          <side-menu-item v-if="showChildren(item)" :key="`menu-${item.name}`" :parent-item="item"></side-menu-item>
          <MenuItem v-else :name="getNameOrHref(item, true)" :key="`menu-${item.children[0].name}`">
            <!-- 图标 -->
            <common-icon :type="item.children[0].icon || ''"></common-icon>
            <span>{{ showTitle(item.children[0]) }}</span>
          </MenuItem>
        </template>
        <template v-else>
          <side-menu-item v-if="showChildren(item)" :key="`menu-${item.name}`" :parent-item="item"></side-menu-item>
          <MenuItem v-else :name="getNameOrHref(item)" :key="`menu-${item.name}`">
            <common-icon :type="item.icon || ''"></common-icon>
            <span>{{ showTitle(item) }}</span>
          </MenuItem>
        </template>
      </template>
    </Menu>
    <!-- 折叠后的菜单 -->
    <div class="menu-collapsed" v-show="collapsed" :list="menuList">
      <template v-for="item in menuList">
        <!-- 如果有子菜单显示子菜单 -->
        <collapsed-menu v-if="item.children && item.children.length > 1" @on-click="handleSelect" hide-title :root-icon-size="rootIconSize" :icon-size="iconSize" :theme="theme" :parent-item="item" :key="`drop-menu-${item.name}`"></collapsed-menu>
        <!-- 否则显示文字提示 -->
        <Tooltip transfer v-else :content="showTitle(item.children && item.children[0] ? item.children[0] : item)" placement="right" :key="`drop-menu-${item.name}`">
          <a @click="handleSelect(getNameOrHref(item, true))" class="drop-menu-a" :style="{textAlign: 'center'}"><common-icon :size="rootIconSize" :color="textColor" :type="item.icon || (item.children && item.children[0].icon)"/></a>
        </Tooltip>
      </template>
    </div>
  </div>
</template>

<script>
import SideMenuItem from './side-menu-item.vue'
import CollapsedMenu from './collapsed-menu.vue'
import { getUnion } from '@/libs/tools'
import mixin from './mixin'

export default {
  name: 'SideMenu',
  mixins: [mixin],
  components: {
    SideMenuItem,
    CollapsedMenu
  },
  props: {
    menuList: {
      type: Array,
      default: () => []
    },
    collapsed: Boolean,
    theme: {
      type: String,
      default: 'dark'
    },
    rootIconSize: {
      type: Number,
      default: 20
    },
    iconSize: {
      type: Number,
      default: 16
    },
    accordion: Boolean, // 是否开启手风琴模式，开启后每次至多展开一个子菜单
    activeName: { // 激活菜单的 name 值
      type: String,
      default: ''
    },
    openNames: { // 展开的 Submenu 的 name 集合
      type: Array,
      defalut: () => []
    }
  },
  data () {
    return {
      openedNames: []
    }
  },
  methods: {
    handleSelect (name) { // 选择菜单（MenuItem）时触发
      this.$emit('on-select', name)
    },
    getOpenedNamesByActiveName (name) {
      return this.$route.matched.map(item => item.name).filter(item => item !== name)
    },
    updateOpenName (name) {
      if (name === this.$config.homeName) this.openedNames = []
      else this.openedNames = this.getOpenedNamesByActiveName(name)
    }
  },
  computed: {
    textColor () {
      return this.theme === 'dark' ? '#fff' : '#495060'
    }
  },
  watch: {
    activeName (name) {
      if (this.accordion) this.openedNames = this.getOpenedNamesByActiveName(name)
      else this.openedNames = getUnion(this.openedNames, this.getOpenedNamesByActiveName(name))
    },
    openNames (newNames) {
      this.openedNames = newNames
    },
    openedNames () {
      this.$nextTick(() => {
        this.$refs.menu.updateOpened()
      })
    }
  },
  mounted () {
    this.openedNames = getUnion(this.openedNames, this.getOpenedNamesByActiveName(name))
  }
}
</script>

<style lang="less" scoped>
.side-menu-wrapper{
  user-select: none;
  .menu-collapsed{
    padding-top: 10px;

    .ivu-dropdown{
      width: 100%;
      .ivu-dropdown-rel a{
        width: 100%;
      }
    }
    .ivu-tooltip{
      width: 100%;
      .ivu-tooltip-rel{
        width: 100%;
      }
      .ivu-tooltip-popper .ivu-tooltip-content{
        .ivu-tooltip-arrow{
          border-right-color: #fff;
        }
        .ivu-tooltip-inner{
          background: #fff;
          color: #495060;
        }
      }
    }
  }
  a.drop-menu-a{
    display: inline-block;
    padding: 6px 15px;
    width: 100%;
    text-align: center;
    color: #495060;
  }
}
.menu-title{
  padding-left: 6px;
}
</style>
