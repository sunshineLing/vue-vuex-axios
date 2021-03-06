<template>
  <div>
    <div class="loader" v-show="loaderShow">
      <svg class="circular" viewBox="25 25 50 50">
        <circle class="path" cx="50" cy="50" r="20" fill="none" stroke-width="5" stroke-miterlimit="10">
        </circle>
      </svg>
    </div>
    <ul class="repoList" v-scroll="loadMore">
      <li v-for="item in list" class="listItem clearfix" :key="item.id">
        <div class="left">
            <h2 class="title">{{item.name}}</h2>
            <p class="description">{{item.description || 'no description'}}</p>
            <p class="updatedTime">updated: {{item.updated_at}}</p>
        </div>
        <div class="right">
          <div class="advatar">
            <img :src="item.owner.avatar_url" width="90" height="90">
          </div>
          <a class="viewMoreLink" :href="item.html_url" target="_blank">view more</a>
        </div>
      </li>
    </ul>
    <div v-if="tipsShow" class="noDataTips">
      <p>sorry, no repository data !</p>
      <a href="https://github.com/new" target="_blank">created new now</a>
    </div>
    <div v-if="subLoaderShow" class="loader listLoad">
      <svg class="circular" viewBox="25 25 50 50">
        <circle class="path" cx="50" cy="50" r="20" fill="none" stroke-width="5" stroke-miterlimit="10">
        </circle>
      </svg>
    </div>
  </div>
</template>

<script>
  import api from '@/common/config/api'
  import * as types from '@/vuex/types'
  import store from '@/vuex/store'
  export default {
    data () {
      return {
        list: [],
        loaderShow: true,
        listPage: 1,
        subLoaderShow: false,
        tipsShow: false
      }
    },
    store,
    mounted () {
      this.getRepository()
    },
    methods: {
      getRepository () {
        let params = {
          page: this.listPage,
          per_page: 10
        }
        //  发起请求
        this.axios.get(api.repo_list, {
          params
        })
        .then(res => {
          if (res.data.length) {
            store.commit(types.TITLE, 'Your repository')
            this.list = res.data
          } else {
            // 显示没有数据的提示
            this.tipsShow = true
          }
          this.loaderShow = false
        })
        .catch(err => {
          console.log(err)
        })
      },
      // 滚动加载
      loadMore () {
        let params = {
          page: ++this.listPage,  // 当前页数
          per_page: 10 // 条数
        }

        // 显示loading
        this.subLoaderShow = true
        this.axios.get(api.repo_list, {
          params
        })
        .then(res => {
          if (res.data.length) {
            this.list = [...this.list, ...res.data]
            store.commit(types.ISBOTTOM, false)
          }
          // 隐藏loadeing
          this.subLoaderShow = false
        })
        .catch(err => {
          console.log(err)
        })
      }
    },
    directives: {
      'scroll': {
        bind (el, binding) {
          window.addEventListener('scroll', evt => {
            if (document.body.scrollTop + window.innerHeight >= el.clientHeight) {
              if (!store.state.isBottom) {
                store.commit(types.ISBOTTOM, true)
                let fn = binding.value
                fn()
              }
            }
          })
        }
      }
    }
  }
</script>

<style lang="scss">
  @import "src/common/style/loader.scss";
  @import "src/common/style/mixin.scss";
  h2 {
    margin: 0;
    padding: 0;
  }
  .repoList {
    position: relative;
    padding: 0;
    margin:0;
    width: 600px;
    margin: 0 auto ;
    list-style: none;
    .listItem {
      position: relative;
      height: 130px;
      padding: 20px;
      box-shadow: 0 0 4px rgba(0,0,0,0.3);
      margin-bottom: 30px;
      transition: all 0.3s;
      background-color: #fff;
      &:hover{
        cursor: pointer;
        box-shadow: 0px 2px 8px 2px rgba(0,0,0,0.2)
      }
    }
    .left{
      float: left;
      width: 80%;
      text-align: left;
      .description{
        color: #888;
      }
      .updatedTime{
        position: absolute;
        bottom: 5px;
      }
    }
    .right{
      float: right;
      width: 20%;
      text-align: right;
      .viewMoreLink{
        display: inline-block;
        width: 100px;
        position: absolute;
        bottom:10px;
        right: 20px;
        font-size: 20px;
        cursor: pointer;
        transition: all 0.5s;
        padding:6px 12px;
        border-radius: 2px;
        text-decoration: none;
        text-align: center;
        color: #000;
        &:hover{
          background-color: hsla(0,0%,60%,.2);
        }
      }
    }
  }
  .listLoad {
      top: -12px;
      width: 20px;
    }
  .noDataTips {
    a {
      @include btn;
    }
  }
</style>
