<template>
  <header @click.self="hideMenu">
    <h1>Minesweeper</h1>
    <div class="menu" @click="toggleMenu">
      <h3>Difficutly level</h3>
      <span class="option" v-if="showOptions" @click="startBeginner">Beginner</span>
      <span class="option" v-if="showOptions" @click="startIntermediate">Intermideate</span>
      <span class="option" v-if="showOptions" @click="startExpert">Expert</span>
    </div>
  </header>
</template>

<script>
export default {
  name: 'Header',
  props: {
    showMenu: Boolean
  },
  data: () => ({
    showOptions: false
  }),
  methods: {
    hideMenu() {
      this.showOptions = false
      this.$emit('hideMenu')
    },
    toggleMenu() {
      this.showOptions = !this.showOptions
      this.$emit('toggleMenu')
    },
    startBeginner() {
      this.$emit('setBeginner')
    },
    startIntermediate() {
      this.$emit('setIntermediate')
    },
    startExpert() {
      this.$emit('setExpert')
    }
  },
  watch: {
    showMenu() {
      this.showOptions = this.showMenu
    }
  }
}
</script>

<style scoped>
header {
  display: flex;
  justify-content: space-between;
}

h1 {
  margin: 10px 20px;
  font-size: 50px;
}

h3 {
  margin: 20px 0 10px 0;
}

.menu {
  margin: 10px 20px 0 0;
  height: 30px;
  width: 200px;
  font-size: 24px;
  cursor: pointer;
  display: flex;
  flex-direction: column;
}

.option {
  padding: 5px;
  margin-top: 5px;
  font-size: 20px;
  background-color: white;
  border-radius: 5px;
  box-shadow: 0 0 5px rgba(0,0,0,0.5);
  transition: 0.2s;
}

.option:hover {
  transform: scale(1.02, 1.02);
  color: black;
}
</style>