<template>
  <el-container style="height: 100vh;">
    <el-aside
      v-show="!contentFullScreen"
      :width="isCollapse ? '60px' : '250px'"
      :class="isCollapse ? 'hide-aside' : 'show-side'"
    >
      <logo-link />
      <menu-text />
    </el-aside>

    <el-container>
      <el-header>
        <header-top />
      </el-header>

      <el-main>
        <router-view v-slot="{ Component, route }">
          <transition
            :name="route.meta.transition || 'fade-transform'"
            mode="out-in"
          >
            <keep-alive
              v-if="keepAliveComponentsName"
              :include="keepAliveComponentsName"
            >
              <component
                :is="Component"
                :key="route.fullPath"
              />
            </keep-alive>
            <component
              :is="Component"
              v-else
              :key="route.fullPath"
            />
          </transition>
        </router-view>
      </el-main>
    </el-container>
  </el-container>
</template>

<script>
import HeaderTop from './components/header/index.vue'
import LogoLink from './components/Logo/index.vue'
import menuText from './components/menu/menu.vue'
import { useEventListener } from '@vueuse/core'
import { defineComponent, computed, onBeforeMount, ref } from 'vue'
import { useStore } from 'vuex'

export default defineComponent({
  components: {
    LogoLink,
    menuText,
    HeaderTop
  },
  setup () {
    const store = useStore()
    // computed
    const isCollapse = computed(() => store.state.app.isCollapse)
    const contentFullScreen = computed(() => store.state.app.contentFullScreen)
    const showLogo = computed(() => store.state.app.showLogo)
    const showTabs = computed(() => store.state.app.showTabs)
    const keepAliveComponentsName = computed(() => store.getters['keepAlive/keepAliveComponentsName'])
    // 页面宽度变化监听后执行的方法
    const resizeHandler = () => {
      if (document.body.clientWidth <= 1000 && !isCollapse.value) {
        store.commit('app/isCollapseChange', true)
      } else if (document.body.clientWidth > 1000 && isCollapse.value) {
        store.commit('app/isCollapseChange', false)
      }
    }
    // 初始化调用
    resizeHandler()
    // beforeMount
    onBeforeMount(() => {
      // 监听页面变化
      useEventListener('resize', resizeHandler)
    })
    // methods
    // 隐藏菜单
    const hideMenu = () => {
      store.commit('app/isCollapseChange', true)
    }

    const tableData = ref([])
    return {
      isCollapse,
      showLogo,
      showTabs,
      contentFullScreen,
      keepAliveComponentsName,
      hideMenu,
      tableData
    }
  }
})
</script>

<style lang="scss" scoped>
.el-header {
  padding-left: 0;
  padding-right: 0;
}
.el-aside {
  display: flex;
  flex-direction: column;
  overflow-x: hidden;
  transition: 0.3s;
  &::-webkit-scrollbar {
    width: 0 !important;
  }
}
.el-main {
  background-color: var(--system-container-background);
  height: 100%;
  padding: 0;
}
.el-main-box {
  width: 100%;
  height: 100%;
  overflow-y: auto;
  box-sizing: border-box;
}
@media screen and (max-width: 1000px) {
  .el-aside {
    position: fixed;
    top: 0;
    left: 0;
    height: 100vh;
    z-index: 1000;
    &.hide-aside {
      left: -250px;
    }
  }
  .mask {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    z-index: 999;
    background: rgba(0, 0, 0, 0.5);
  }
}
</style>
