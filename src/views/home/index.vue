<template>
    <div class="home">
      <header class="header con">
        <div class="header-top">
          <div class="search-con">
            <search-box @focus="handleFocus" />
          </div>
          <div class="tag-setting-con">
            <router-link to="/tag-manage">
              <i class="iconfont icon-shezhi"></i>
              <span>标签</span>
            </router-link>
          </div>
        </div>
        <div class="header-bottom">
          <div class="nav-tab-con">
            <nav-tab :nav-list="routeList"></nav-tab>
          </div>
          <div class="triangle-con">
            <div @click="goSpecialEdit" class="triangle"></div>
          </div>
        </div>
      </header>
      <section class="main">
        <router-transition prefix="/home">
          <keep-alive>
            <router-view :key="routeKey" class="router"></router-view>
          </keep-alive>
        </router-transition>
        <div class="icon-add-con">
          <i class="iconfont icon-Add"></i>
        </div>
      </section>
    </div>
</template>

<script>
import SearchBox from '../../components/search-box'
import NavTab from '../../components/nav-tab'
import { defaultHomeRoutes } from '../../common/config'
import { mapState } from 'vuex'
let path = window.location.pathname
path = path.startsWith('/home/') ? path : '/home/recommended'
export default {
  components: {
    SearchBox,
    NavTab,
  },
  data() {
    return {
      routeKey: path
    }
  },
  computed: {
    
    ...mapState({
      routeList(state) {
        return defaultHomeRoutes.concat(state.homeRoutes.filter(item => item.show))
      }
    }),
    
  },
  created() {
    
  },
  activated() {
    
  },
  deactivated() {
    
  },
  methods: {
    handleFocus(e) {
      e.target.blur()
      this.$router.push('/search')
    },
    goSpecialEdit() {
      this.$router.push({
        name: 'specialShowEdit',
        params: {
          stateKey: 'homeRoutes'
        }
      })
    }
  },
  watch: {
    $route() {
      let { path } = this.$route
      if (path.startsWith('/home/')) {
        this.routeKey = path
      }
    }
  }
}
</script>

<style lang="stylus" scoped>
@import "../../assets/css/variable.styl"
.home
  height 100%
  position relative
.header
  display flex
  flex-direction column
  justify-content space-between
  height 148*$unit
  background $primary-color
  .header-top
    display flex
    padding-top 15*$unit 
    height 75*$unit
    .search-con
      flex 1
      height 100%
    .tag-setting-con
      flex 0 0 120*$unit
      display flex
      align-items center
      justify-content space-around
      padding-left 20*$unit
      height 100%
      color #fff
      font-size 30*$unit
      .iconfont
        font-size 30*$unit
  .header-bottom
    display flex
    align-items center
    width 100%
    .nav-tab-con
      flex 1
      overflow hidden
      height 75*$unit
    .triangle-con
      padding-left: 30*$unit
      flex 0 0 30*$unit
      .triangle
        width 0
        height 0
        border: 15*$unit solid transparent
        border-top: 15*$unit solid #fff
.main
  position absolute
  top 148*$unit
  bottom 0
  width 100%
  .icon-add-con
    position fixed
    right 30*$unit
    bottom 120*$unit
    border-radius 50%
    width 110*$unit
    height 110*$unit
    background $primary-color
    text-align center
    color #fff
    line-height 110*$unit
    z-index 12
    .iconfont
      font-size 30*$unit
</style>