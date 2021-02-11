<template>
  <div class="wrapper-content wrapper-content--fixed">

      <section>

        <div class="container">

          <div class="error"  v-if="error" style="margin-bottom:20px">
            <p>{{ error }}</p>
          </div>

          <Search 
            :value="search"
            placeholder="Type username..."
            @search="search = $event"
          />

          <button v-if="!repos" class="btn btnPrimary" @click="getRepos">Search!</button>
          <button v-else class="btn btnPrimary" @click="getRepos">Search again!</button>

          <div class="user__wrapper">
            <div class="user-avatar" v-if="user.avatar">
              <img :src="user.avatar" alt="User avatar">
            </div>
            <div class="user-name" v-if="user.name">
              <p>{{user.name}}</p>
            </div>
            <div class="user-count" v-if="user.reposCount">
              <p>Repos count: <span>{{user.reposCount}}</span></p>
            </div>
          </div>

          <div class="repos__wrapper" v-if="repos">
            <div class="repos-title">
              <div class="repo-name" @click="sort('name')">
                Name <span class="sort-arrow" v-bind:class="[currentSort === 'name' ? 'active': '', currentSortDir]"></span>
              </div>
              <div class="repo-stars" @click="sort('stargazers_count')">
                Stars <span class="sort-arrow" v-bind:class="[currentSort === 'stargazers_count' ? 'active': '', currentSortDir]"></span>
              </div>
            </div>
            <div class="repo-item" v-for="repo in reposSort" :key="repo.id">
              <div class="repo-info">
                <a class="link" target="_blank" :href="repo.html_url">{{ repo.name }}</a>
                <span> {{ repo.stargazers_count }} ⭐</span>
              </div>
            </div>
          </div>

        </div>

      </section>

      <section>
        <div class="container">
          <div class="button-list" v-if="reposCount">
            <div class="btn btnPrimary" @click="prevPage">&#8592;</div>
            <span>{{ page.current }}/{{ reposCount }}</span>
            <div class="btn btnPrimary" @click="nextPage">&#8594;</div>
          </div>
        </div>
      </section>

    </div>
</template>

<script>
import Search from '@/components/Search.vue'
export default {
  components: {Search},
  data() {
    return {
      search: '',
      repos: null,
      error: null,
      user: {
        avatar: null,
        name: null,
        reposCount: null
      },
      currentSort: 'name',
      currentSortDir: 'asc',
      page: {
        current: 1,
        length: 4,
        count: null
      }
    }
  },
  computed: {
    reposSort() {
      return this.repos.sort((a, b) => {
        let mod = 1
        if (this.currentSortDir === 'desc') mod = -1
        if (a[this.currentSort] < b[this.currentSort]) return -1 * mod
        if (a[this.currentSort] > b[this.currentSort]) return 1 * mod
        return 0
      }).filter( (row, index) => {
        let start = (this.page.current - 1) * this.page.length
        let end = this.page.length * this.page.current
        if (index >= start && index < end) return true
      })
    },
    reposCount() {
      return Math.ceil(this.repos?.length/this.page.length)
    },
  },
  methods: {
    getRepos() {
      const urls = [
        `https://api.github.com/users/${this.search}/repos`,
        `https://api.github.com/users/${this.search}`
      ]
      Promise.all(urls.map((anUrl) => fetch(anUrl)))
      .then(responses => 
        Promise.all(responses.map( (res) => {
              if (res.ok) {
              this.error = null
              return res.json()
            } else {
              this.repos = null
              this.error = 'Can`t find this user'
              return false
            }
          }))
      ).then(results => {
        this.user.avatar = results[1].avatar_url
        this.user.name = results[1].name
        this.user.reposCount = results[1].public_repos
        this.repos = results[0]
      })
    },
    sort(e) {
      if (e == this.currentSort) {
        this.currentSortDir = this.currentSortDir === 'asc' ? 'desc' : 'asc'
      }
      else {
        this.currentSort = e
        this.currentSortDir = 'asc'
      }
    },
    prevPage() {
      if (this.page.current > 1) {
        this.page.current -= 1
      }
     },
    nextPage() {
      if( (this.page.current * this.page.length) < this.repos.length) {
        this.page.current += 1
      } 
    }
  }
}
</script>

<style lang="scss" scoped>
  .container{
    display: flex;
    align-items: center;
    flex-direction: column;
  }
  button{
    margin-top: 2em;
  }
  .repos__wrapper{
    width: 400px;
    margin: 30px 0;
  }
  .repo-info{
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
    padding: 10px 0;
    border-bottom: 1px solid #dbdbdb;
  }
  .repos-title{
    display: flex;
    align-items: center;
    justify-content: space-between;
    margin-bottom: 10px;
    padding: 10px 0;
    border-bottom: 1px solid #dbdbdb;
  }
  .user__wrapper{
    display: flex;
    align-items: center;
    margin: 2em 0;
    .user-avatar{
      margin:0 1em;
      img{
        max-width: 3em;
        max-height: 3em;
      }
    }
    .user-name{
      margin:0 1em;
      font-weight: 600;
    }
    .user-count{
      margin:0 1em;
      color:#8d8d8d;
      span{
        color:#525252;
      }
    }
  }
  .sort-arrow{
    &:before{
      content: '↓';
      color: #ccc;
    }
    &.active:before{
      color: #000;
    }
    &.active.desc:before{
      content: '↑';
    }
  }
  .button-list{
    text-align: center;
    width: 100%;
    .btn {
      border-radius: 50px;
      margin:0 20px;
    }
  }
</style>
