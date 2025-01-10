<template>
  <div class="controls-container">
    <div class="controls">
      <div class="trade-controls">
        <div class="balance">
          <label for="timeframe">Balance: {{ balance }}$</label>
        </div>
        <div class="bet-controls">
          <div>
            <label for="timeframe">Timeframe:</label>
            <select id="timeframe" v-model="timeframe" @change="saveTimeframe">
              <option value="10">10 seconds</option>
              <option value="30">30 seconds</option>
              <option value="60">1 minute</option>
              <option value="180">3 minutes</option>
            </select>
          </div>
          <div>
            <label for="bet">Bet:</label>
            <input type="number" id="bet" v-model="bet" min="1" step="0.01" @input="formatBet" @change="saveBet" />
          </div>
        </div>
      </div>
      <div class="buttons">
        <button class="call-button" @click="call" :disabled="balance < bet">Call</button>
        <button class="put-button" @click="put" :disabled="balance < bet">Put</button>
      </div>
    </div>
  </div>
</template>

<script>
import { openDB } from 'idb';
import c_bet from "@/consts/bet.js";
import c_props from "@/consts/props.js";
import c_storage from "@/consts/storage.js";

export default {
  props: [c_props.Balance],
  data() {
    return {
      timeframe: 60,
      bet: 10
    };
  },
  async created() {
    this.db = await openDB(c_storage.DBName, 1, {
      upgrade(db) {
        db.createObjectStore(c_storage.CollectionName, { keyPath: 'id' });
      },
    });

    const savedTimeframe = await this.db.get(c_storage.CollectionName, c_storage.TimeframeKey);
    const savedBet = await this.db.get(c_storage.CollectionName, c_storage.BetAmountKey);

    if (savedTimeframe) {
      this.timeframe = savedTimeframe.value;
    }

    if (savedBet) {
      this.bet = savedBet.value;
    }
  },
  methods: {
    formatBet() {
      const value = +parseFloat(this.bet).toFixed(2).replace(/\.00$/, '');
      this.bet = isNaN(value) ? 0 : value;
    },
    async saveTimeframe() {
      await this.db.put(c_storage.CollectionName, { id: c_storage.TimeframeKey, value: this.timeframe });
    },
    async saveBet() {
      await this.db.put(c_storage.CollectionName, { id: c_storage.BetAmountKey, value: this.bet });
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
  flex-wrap: wrap;
  justify-content: center;
  align-items: center;
  gap: 10px;
}

.bet-controls {
  display: flex;
  flex-direction: row;
  gap: 10px;
  align-items: center;
}

.trade-controls {
  display: flex;
  flex-direction: row;
  gap: 10px;
}

.balance {
  display: flex;
  justify-content: center;
  align-items: center;
  border-radius: 4px;
  padding: 5px;
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
  padding: 3px;
  max-width: 100px;
}

select:focus,
input:focus {
  outline: none;
  border-color: #1e88e5;
}

.buttons {
  display: flex;
  gap: 10px;
  width: 100%;
  justify-content: center;
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

@media (max-width: 600px) {
  .controls {
    flex-direction: column;
  }

  .buttons {
    flex-direction: row;
  }

  .bet-controls {
    gap: 1px;
  }

  .controls-container {
    padding-top: 15px;
  }
}
</style>