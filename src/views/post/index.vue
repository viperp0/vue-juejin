<template>
  <div class="post" >
    <header class="header">
      <d-header title="文章详情页">
        <div ref="header-con" class="header-con">
          <div v-if="entry" ref="header-content" class="header-content">
            <div class="user-con">
              <s-author :author="entry.user"></s-author>
            </div>
            <div class="title-con">
              <h2 class="title">文章详情页</h2>
            </div>
          </div>
        </div>
      </d-header>
    </header>
    <load-scroll @scroll="postScroll" :load-more="loadMore" :load-finish="loadFinish" :is-loading="isLoading">
      <div>
        <section class="article">
          <div v-if="entry" class="summary">
            <div class="author-container">
              <author :author="entry.user"></author>
            </div>
            <h1 class="title">{{entry.title}}</h1>
            <a class="origin-url" target="_blank" :href="entry.originalUrl">原文链接</a>
            <div class="screenshot" v-if="entry.screenshot" :style="{backgroundImage: `url(${entry.screenshot})`}"></div>
          </div>
          <div class="article-content" ref="post" v-html="content"></div>
          <div class="tag-con">
            <ul class="tag-list" v-if="entry && entry.tags">
              <li class="tag-item" v-for="item in entry.tags" :key="item.id">
                {{item.title}}
              </li>
            </ul>
          </div>
          <div v-if="entry" class="article-info">
            阅读 {{entry.viewsCount}} · 赞 {{entry.collectionCount}}
          </div>
        </section>
        <section class="related-article con">
          <h3 v-if="entry" class="title border-bottom-1px">{{entry.user.username}}的更多文章</h3>
          <ul class="related-article-list">
            <li class="related-article-item border-bottom-1px" v-for="item in relatedEntry" :key="item.objectId">
              <router-link :to="`/post/${item.id || item.objectId}`">
                <article-s-entry :article="item"></article-s-entry>
              </router-link>
            </li>
          </ul>
        </section>
        <section class="comment-con con">
          <ul v-if="comments.length>0" class="comment-list">
            <li v-for="item in comments" class="comment-item border-bottom-1px" :key="item.id">
              <comment :authorId="entry.user.id" :comment="item"></comment>
            </li>
          </ul>
          <div v-else class="no-comment">
            <i class="iconfont icon-Comment"></i>
            <span>暂无评论</span>
          </div>
        </section>
      </div>
    </load-scroll>
  </div>
</template>

<script>
import { getEntryView, getEntryByEntryIds, getRelatedEntry, getComments } from '../../api/post'

import DHeader from '../../components/d-header'
import ArticleSEntry from '../../components/article-s-entry'
import Comment from '../../components/comment'
import Loading from '../../components/loading'
import Author from '../../components/author'
import SAuthor from '../../components/s-author'
import LoadScroll from '../../components/load-scroll'

export default {
  name: 'post',
  data() {
    return {
      content: '',
      entry: null,
      relatedEntry: [],
      comments: [],
      isLoading: false,
      loadFinish: false
    }
  },
  components: {
    DHeader,
    ArticleSEntry,
    Comment,
    Loading,
    Author,
    LoadScroll,
    SAuthor
  },
  created() {
    this.init()
  },
  mounted() {
    this.headerConHeight = this.$refs['header-con'] && this.$refs['header-con'].clientHeight
    this.$headerContent = this.$refs['header-content']
  },
  updated() {
    this.headerConHeight = this.$refs['header-con'].clientHeight && this.$refs['header-con'].clientHeight
    this.$headerContent = this.$refs['header-content']
  },
  methods: {
    init() {
      let { id } = this.$route.params
      this.postId = id
      this.getEntryByEntryIds(id)
      this.getRelatedEntry(id)
      this.getEntryView(id)
      this.getComments(id)
    },
    follow() {
      // TODO
    },
    postScroll(e) {
      let { scrollTop } = e.target
      if (scrollTop > this.headerConHeight) {
        this.$headerContent.style.transform = `translateY(0)`
      } else {
        this.$headerContent.style.transform = `translateY(-50%)`
      }
    },
    async loadMore() {
      if(!this.isLoading) {
        let { createdAt  }= this.comments[this.comments.length - 1]

        this.isLoading = true
        let data = await getComments(this.postId, createdAt)
        if(data.d.comments.length === 0) {
          this.loadFinish = true
          setTimeout(() => {
            this.isLoading = false
          }, 500)
          return 
        }
        data.d.comments.forEach(item => {
          this.comments.push(item)
        })
        this.isLoading = false
      }
    },
    async getEntryView(id) {
      let data = await getEntryView(id)
      this.content = data.d.content
    },
    async getEntryByEntryIds(id) {
      let data = await getEntryByEntryIds(id)
      let entry = data.d.entrylist[0]
      let user = entry.user
      this.entry = {
        title: entry.title,
        screenshot: entry.screenshot,
        originalUrl: entry.originalUrl,
        tags: entry.tags,
        collectionCount: entry.collectionCount,
        viewsCount: entry.viewsCount,
        user: {
          id: user.objectId,
          username: user.username,
          jobTitle: user.jobTitle,
          company: user.company,
          level: user.level,
          avatarLarge: user.avatarLarge
        }
      }
    },
    async getRelatedEntry(id) {
      let data = await getRelatedEntry(id)
      this.relatedEntry = data.d.entrylist.map(item => ({
        id: item.objectId,
        title: item.title,
        collectionCount: item.collectionCount,
        screenshot: item.screenshot,
        user: {
          username: item.user.username
        }
      }))
    },
    async getComments(id, created) {
      let data = await getComments(id, created)
      this.comments = data.d.comments
      if (this.comments.length === 0) {
        this.loadFinish = true
      }
    }
  }
}
</script>

<style lang="stylus" scoped>
.post
  background #f4f8fb
  .header
    position fixed
    width 100%
    border-bottom 1px solid #edeeef
    background #fff
    z-index 1
    .header-con
      height 100%
      width 100%
      overflow hidden
      .header-content
        transform translateY(-50%)
        transition transform .5s
        .user-con
          height 105rem
          display flex
          align-items center
          justify-content space-between
          .user-info
            display flex
            align-items center
            .avatar
              width 60rem
              height 60rem
          .userinfo
            display flex
            align-items center
            .username
              margin-right 8rem
              font-size 25rem
            .level
              height 25rem
        .title-con
          height 105rem
          line-height 105rem
      .follow
        display flex
        justify-content center
        align-items center
        flex 0 0 140rem
        height 60rem
        border 1px #6cbd45 solid
        color #6cbd45
        font-size 26rem
        .iconfont
          padding 0
          margin-right 10rem
          color #6cbd45
          font-size 26rem
  .article
    margin-bottom 20rem
    padding 125rem 20rem 50rem
    background #fff
    .summary
      .author-container
        margin-bottom 20rem
      .title
        font-size 35rem
        margin-bottom 45rem
      .origin-url
        margin-bottom 30rem
        font-size 25rem
        color #909090
      .screenshot
        margin 20rem 0
        background-size cover
        background-position 50%
        background-repeat no-repeat
        &::after
          content: ""
          display: block
          padding-top: 56.29%
          width: 100%
          pointer-events: none
    .article-content
      margin-bottom 65rem
    .tag-con
      margin-bottom 50rem
    .tag-list
      display flex
      .tag-item
        padding 15rem 20rem
        background #ebebeb
        color #666
        margin-right 10rem
        font-size 20rem
    .article-info
      color #909090
      font-size 25rem
      line-height 1

  .related-article
    margin-bottom 20rem
    background #fff
    .title
      height 100rem
      line-height 100rem
      font-size 30rem
      font-weight 500
      color #333
  .loading-con
    display flex
    justify-content center
    align-items center
    height 100rem
    background-color #fff
    font-size 25rem
    color #666
.comment-con
  .no-comment
    height 300rem
    background: #fff
    display flex
    flex-direction column
    justify-content center
    align-items center
    font-size 30rem
    color #909090
    .iconfont
      font-size 70rem
</style>