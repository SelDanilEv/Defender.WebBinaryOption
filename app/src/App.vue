<template>
  <div id="app">
    <header>
      <h1>Binary Options Trading <br>
        (2023 Real Data)</h1>
      <button class="reset-button" @click="resetBalance">Reset</button>
    </header>
    <main>
      <Game ref="gameComponent" />
    </main>
  </div>
</template>

<script>
import Game from "@/views/BinaryOptionGame.vue";

export default {
  name: "App",
  components: {
    Game,
  },
  methods: {
    resetBalance() {
      this.$refs.gameComponent.resetBalance();
    }
  }
};
</script>

<style scoped>
#app {
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  background-color: #121212;
  color: #ffffff;
  font-family: Arial, sans-serif;
  margin: 0;
}

header {
  display: flex;
  text-align: center;
  justify-content: space-between;
  align-items: center;
  background-color: #1e88e5;
  padding: 20px;
  color: white;
  position: relative;
}

header h1 {
  margin: 0;
  font-size: 1.8rem;
  flex: 1 0 70%;
}

.reset-button {
  flex: 0 0 10%;
  background-color: #757575;
  color: white;
  border: none;
  border-radius: 4px;
  padding: 10px 20px;
  cursor: pointer;
  text-align: center;
}

.reset-button:hover {
  background-color: #616161;
}

main {
  flex: 1;
  padding: 20px;
  display: flex;
  justify-content: center;
  align-items: top;
}

@media (max-width: 600px) {
  header {
    padding: 10px;
  }

  header h1 {
    font-size: 1.2rem;
  }

  .reset-button {
    flex: 0 0 20%;
  }

  main {
    padding: 0px;
  }
}
</style>