<template>
  <div>
    <van-nav-bar class="navbar">
      <template #title>
        <van-button
          class="search-btn"
          round
          icon="search"
          @click="$router.push('/search')"
        >
          搜索</van-button
        >
      </template>
    </van-nav-bar>
    <!-- 文章 -->
    <van-tabs v-model="active" swipeable>
      <van-tab v-for="item in myChannels" :key="item.id" :title="item.name"
        ><article-list :id="item.id"></article-list
      ></van-tab>
      <span class="toutiao toutiao-gengduo1" @click="show = true"> </span>
      <!-- 频道弹层 -->
      <van-popup
        v-model="show"
        closeable
        position="bottom"
        close-icon-position="top-left"
        :style="{ height: '100%' }"
      >
        <!-- $event模板
          原生事件：事件对象
          自定义事件当中：于组件传递过来的第一个参数 -->
        <channel-popup
          :channels="myChannels"
          @close="show = false"
          @change-active="active = $event"
          @delChannel="delChannelfn"
          @add-channel="addChannel"
        ></channel-popup>
      </van-popup>
    </van-tabs>
  </div>
</template>

<script>
import ArticleList from './components/ArticeList.vue'
import ChannelPopup from './components/ChannelPopup.vue'
import {
  getMyChannel as getMyChannelAPI,
  delChannel as delChannelAPI,
  addChannel,
  setMyChannelToLocal,
  getMyChannelByLocal
} from '@/api'
export default {
  name: 'Home',
  data() {
    return {
      myChannels: [],
      active: 0,
      show: false
    }
  },
  components: {
    ArticleList,
    ChannelPopup
  },
  computed: {
    islogin() {
      return !!this.$store.state.tokenObj.token
    }
  },
  created() {
    this.initMyChannel()
  },
  methods: {
    initMyChannel() {
      if (this.isLogin) {
        // 用户登录了
        //  - 发送请请求获取mycahnnels数据
        this.getMyChannel()
      } else {
        // 用户未登录
        const myChannels = getMyChannelByLocal()

        if (myChannels) {
          //  - 本地存储有mycahnnels数据, 从本地拿myChannels
          this.myChannels = myChannels
        } else {
          //  - 本地存储没有mycahnnels数据, 获取默认的mychannels
          this.getMyChannel()
        }
      }
    },

    // 加载
    loading() {
      this.$toast.loading({
        message: '加载中',
        forbidClick: true
      })
    },
    //获取我的频道列表请求
    async getMyChannel() {
      this.loading()
      try {
        const res = await getMyChannelAPI()
        this.myChannels = res.data.data.channels
        // this.$toast.resetDefaultOptions({ duration: 1000 })
        // console.dir(this.$toast.success)
        // this.$toast.success('获取成功')
      } catch (e) {
        console.log(e)
        this.$toast.fail('获取频道失败，请刷新')
      }
    },
    //添加推荐频道到我的频道
    async addChannel(channel) {
      this.loading()
      try {
        if (this.islogin) {
          //添加到服务器
          await addChannel(channel.id, this.myChannels.length)
        } else {
          setMyChannelToLocal([...this.myChannels, channel])
        }
        //添加到页面
        this.myChannels.push(channel)
        this.$toast.success('添加频道成功')
      } catch (e) {
        console.log(e)
        this.$toast.fail('添加频道失败')
      }
    },
    //删除点击的频道
    async delChannelfn(id) {
      this.loading()
      //删除后的频道列表
      const newChannels = this.myChannels.filter((item) => item.id !== id)
      try {
        if (this.islogin) {
          //删除服务器数据
          await delChannelAPI(id)
        } else {
          setMyChannelToLocal(newChannels)
        }
        this.myChannels = newChannels
        // //删除本地数据
        // this.myChannels.splice(
        //   this.myChannels.findIndex((it) => it.id === id),
        //   1
        // )
        this.$toast.success('删除成功')
      } catch (e) {
        console.dir(e)
        this.$toast.fail('删除失败')
      }
    }
  }
}
</script>

<style scoped lang="less">
.navbar {
  background-color: #3296fa;
  :deep(.van-nav-bar__title) {
    max-width: unset;
  }

  :deep(.van-icon) {
    color: #fff;
  }
  position: sticky;
  top: 0;
  left: 0;
}

.search-btn {
  width: 7.4rem;
  height: 0.85333rem;
  background-color: #5babfb;
  color: #fff;

  &.van-button--default {
    border: 0.02667rem solid #5babfb;
  }
}
.van-tab__pane {
  // 符号的两侧必须由空格
  min-height: calc(100vh - 92px - 88px - 100px);
}
/* tabs导航条样式 */
:deep(.van-tabs__wrap) {
  padding-right: 66px;
  position: sticky;
  top: 92px;
  left: 0;
  z-index: 999;
  .van-tabs__nav {
    padding-left: 0;
    padding-right: 0;

    /* tab标签 */
    .van-tab {
      border: 1px solid #eee;
      width: 200px;
      height: 82px;
    }

    /* tab标签下划线 */
    .van-tabs__line {
      width: 31px;
      height: 6px;
      background-color: #3296fa;
      bottom: 40px;
    }
  }
}

/* 字体图标 */
.toutiao-gengduo1 {
  position: absolute;
  top: 0;
  right: 0;
  width: 66px;
  height: 82px;
  font-size: 40px;
  line-height: 82px;
  text-align: center;
  opacity: 0.6;
  border-bottom: 1px solid #eee;

  position: fixed;
  top: 92px;
  right: 0;
  z-index: 999;

  &::after {
    content: '';
    position: absolute;
    left: 0;
    top: 50%;
    transform: translateY(-50%);
    height: 70%;
    width: 1px;
    background-image: url('~@/assets/images/gradient-gray-line.png');
  }
}
:deep(.van-tabs__content) {
  // 符号的两侧必须由空格
  max-height: calc(100vh - 92px - 88px - 100px);
  padding-bottom: 100px;
  // 溢出时候 如果内容被修剪，则浏览器会显示滚动条
  overflow: auto;
}
</style>
