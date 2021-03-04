<template>
  <Header :showMenu="showMenu"
          @hideMenu="hideMenu"
          @toggleMenu="toggleMenu"
          @setBeginner="setBeginner"
          @setIntermediate="setIntermediate"
          @setExpert="setExpert"
  />
  <main @click="hideMenu">
    <Play :difficulty="difficulty" />
  </main>
</template>

<script>
import Header from './components/Header'
import Play from './components/Play'

export default {
  name: 'App',
  components: {
    Header,
    Play
  },
  data: () => ({
    difficulty: 'beginner',
    showMenu: false
  }),
  methods: {
    toggleMenu() {
      this.showMenu = !this.showMenu
    },
    hideMenu() {
      this.showMenu = false
    },
    setBeginner() {
      this.difficulty = 'beginner'
    },
    setIntermediate() {
      this.difficulty = 'intermediate'
    },
    setExpert() {
      this.difficulty = 'expert'
    }
  }
}
</script>

<style>
body {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  color: #344469;
  background-color: #eee;

  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
}

main {
  padding-top: 50px;
  display: flex;
  justify-content: space-around;
}
</style>
