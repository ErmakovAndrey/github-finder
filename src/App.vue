<template>

  <div class="wrapper">

    <header>
      <div class="navbar">
        <div class="container">
          <div class="navbar-content">
            <div class="logo">
              GitHub-Finder
            </div>
          </div>
        </div>
      </div>
    </header>

    <router-view></router-view>

    <footer></footer>

  </div>

</template>

<script>
import Search from './components/Search.vue'

export default {
  data() {
    return {
      links: [
        {
          title: 'Home',
          url: '/'
        }
      ]
    }
  }
}
</script>

<style lang="scss">
.navbar-link{
  &.router-link-exact-active{
    color: rgb(252, 0, 0);
  }
}
  
</style>
