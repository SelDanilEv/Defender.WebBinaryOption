<template>
  <div class="controls-container">
    <div class="controls">
      <div style="width: 150px;">
        <label for="timeframe">Balance: {{ balance }}$</label>
      </div>
      <div>
        <label for="timeframe">Timeframe:</label>
        <select id="timeframe" v-model="timeframe">
          <option value="10">10 seconds</option>
          <option value="30">30 seconds</option>
          <option value="60">1 minute</option>
          <option value="180">3 minutes</option>
        </select>
      </div>
      <div>
        <label for="bet">Bet:</label>
        <input type="number" id="bet" v-model="bet" min="1" step="0.01" @input="formatBet" />
      </div>
      <div class="buttons">
        <button class="call-button" @click="call" :disabled="balance < bet">Call</button>
        <button class="put-button" @click="put" :disabled="balance < bet">Put</button>
      </div>
    </div>
  </div>
</template>

<script>
import c_bet from "@/consts/bet.js";
import c_props from "@/consts/props.js";

export default {
  props: [c_props.Balance],
  data() {
    return {
      timeframe: 60,
      bet: 10
    };
  },
  methods: {
    formatBet() {
      const value = +parseFloat(this.bet).toFixed(2).replace(/\.00$/, '');
      this.bet = isNaN(value) ? 0 : value;
    },
    call() {
      this.$emit("action", { type: c_bet.CallType, timeframe: this.timeframe, bet: this.bet });
    },
    put() {
      this.$emit("action", { type: c_bet.PutType, timeframe: this.timeframe, bet: this.bet });
    }
  }
};
</script>

<style scoped>
.controls-container {
  display: flex;
  justify-content: center;
  height: fit-content;
}

.controls {
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  gap: 10px;
}

label {
  margin-right: 10px;
  color: #ffffff;
}

select,
input {
  background-color: #333333;
  color: #ffffff;
  border: 1px solid #444444;
  border-radius: 4px;
  padding: 5px;
}

select:focus,
input:focus {
  outline: none;
  border-color: #1e88e5;
}

.buttons {
  display: flex;
  gap: 10px;
}

.call-button {
  background-color: green;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.put-button {
  background-color: red;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.call-button:disabled,
.put-button:disabled {
  background-color: grey;
  cursor: not-allowed;
}

.call-button:hover:not(:disabled) {
  background-color: darkgreen;
}

.put-button:hover:not(:disabled) {
  background-color: darkred;
}
</style>