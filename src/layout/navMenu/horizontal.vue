<template>
  <div class="el-menu-horizontal-warp">
    <el-scrollbar
      @wheel.native.prevent="onElMenuHorizontalScroll"
      ref="elMenuHorizontalScrollRef"
    >
      <el-menu
        router
        :default-active="defaultActive"
        :ellipsis="false"
        background-color="transparent"
        mode="horizontal"
      >
        <template v-for="val in menuLists">
          <el-sub-menu
            :index="val.path"
            v-if="val.children && val.children.length > 0"
            :key="val.path"
          >
            <template #title>
              <SvgIcon :name="val.meta.icon" />
              <span>{{ $t(val.meta.title) }}</span>
            </template>
            <SubItem :chil="val.children" />
          </el-sub-menu>
          <el-menu-item :index="val.path" :key="val.path" v-else>
            <template
              #title
              v-if="!val.meta.isLink || (val.meta.isLink && val.meta.isIframe)"
            >
              <SvgIcon :name="val.meta.icon" />
              {{ $t(val.meta.title) }}
            </template>
            <template #title v-else>
              <a :href="val.meta.isLink" target="_blank" rel="opener">
                <SvgIcon :name="val.meta.icon" />
                {{ $t(val.meta.title) }}
              </a>
            </template>
          </el-menu-item>
        </template>
      </el-menu>
    </el-scrollbar>
  </div>
</template>

<script lang="ts">
import {
  toRefs,
  reactive,
  computed,
  defineComponent,
  getCurrentInstance,
  onMounted,
  nextTick,
  onBeforeMount,
} from "vue";
import { useRoute, onBeforeRouteUpdate } from "vue-router";
import { useThemeConfigStateStore } from "@/stores/themeConfig";
import { useRoutesListStore } from "@/stores/routesList";
import SubItem from "@/layout/navMenu/subItem.vue";
export default defineComponent({
  name: "navMenuHorizontal",
  components: { SubItem },
  props: {
    menuList: {
      type: Array,
      default: () => [],
    },
  },
  setup(props) {
    const { proxy } = getCurrentInstance() as any;
    const route = useRoute();
     
    const theme = useThemeConfigStateStore();
    const routesList = useRoutesListStore();
    const state: any = reactive({
      defaultActive: null,
    });
    // ????????????????????????
    const menuLists = computed(() => {
      return props.menuList;
    });
    // ?????????????????????????????????????????????
    const onElMenuHorizontalScroll = (e: any) => {
      const eventDelta = e.wheelDelta || -e.deltaY * 40;
      proxy.$refs.elMenuHorizontalScrollRef.$refs.wrap.scrollLeft =
        proxy.$refs.elMenuHorizontalScrollRef.$refs.wrap.scrollLeft + eventDelta / 4;
    };
    // ??????????????????????????????????????????????????????????????????
    // ??????????????????????????????????????????????????????????????????
    const initElMenuOffsetLeft = () => {
      nextTick(() => {
        let els: any = document.querySelector('.el-menu.el-menu--horizontal li.is-active');
        if (!els) return false;
        proxy.$refs.elMenuHorizontalScrollRef.$refs.wrap$.scrollLeft = els.offsetLeft;
      });
    };
    // ????????????????????????
    const filterRoutesFun = (arr: Array<object>) => {
      return arr
        .filter((item: any) => !item.meta.isHide)
        .map((item: any) => {
          item = Object.assign({}, item);
          if (item.children) item.children = filterRoutesFun(item.children);
          return item;
        });
    };
    // ????????????????????????????????????
    const setSendClassicChildren = (path: string) => {
      const currentPathSplit = path.split("/");
      let currentData: any = {};
      filterRoutesFun(routesList.routesList).map((v, k) => {
        if (v.path === `/${currentPathSplit[1]}`) {
          v["k"] = k;
          currentData["item"] = [{ ...v }];
          currentData["children"] = [{ ...v }];
          if (v.children) currentData["children"] = v.children;
        }
      });
      return currentData;
    };
    // ??????????????????????????????
    const setCurrentRouterHighlight = (currentRoute:any) => {
      const { path, meta } = currentRoute;
      if (theme.themeConfig.layout === "classic") {
        state.defaultActive = `/${path.split("/")[1]}`;
      } else {
        const pathSplit = meta.isDynamic
          ? meta.isDynamicPath.split("/")
          : path.split("/");
        if (pathSplit.length >= 4 && meta.isHide)
          state.defaultActive = pathSplit.splice(0, 3).join("/");
        else state.defaultActive = path;
      }
    };
    // ???????????????
    onBeforeMount(() => {
      setCurrentRouterHighlight(route);
    });
    // ???????????????
    onMounted(() => {
      initElMenuOffsetLeft();
    });
    // ???????????????
    onBeforeRouteUpdate((to) => {
      // ?????????https://gitee.com/PandaAdmin/PandaX/issues/I3YX6G
      setCurrentRouterHighlight(to);
      proxy.mittBus.emit("onMenuClick");
      // ????????????????????????????????????????????????tagsView??????????????????????????????????????????
      let { layout, isClassicSplitMenu } = theme.themeConfig;
      if (layout === "classic" && isClassicSplitMenu) {
        proxy.mittBus.emit("setSendClassicChildren", setSendClassicChildren(to.path));
      }
    });
    return {
      menuLists,
      onElMenuHorizontalScroll,
      ...toRefs(state),
    };
  },
});
</script>

<style scoped lang="scss">
.el-menu-horizontal-warp {
  flex: 1;
  overflow: hidden;
  margin-right: 30px;
  ::v-deep(.el-scrollbar__bar.is-vertical) {
    display: none;
  }
  ::v-deep(a) {
    width: 100%;
  }
  .el-menu.el-menu--horizontal {
    display: flex;
    height: 100%;
    width: 100%;
    box-sizing: border-box;
  }
}
</style>
