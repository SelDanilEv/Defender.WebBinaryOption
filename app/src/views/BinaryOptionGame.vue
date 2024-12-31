<template>
  <div style="width: 90%">
    <Controls :balance="balance" @action="handleAction" />
    <Chart :data="displayedRates" :bets="bets" />
    <OptionBetsList :bets="bets" :current-index="currentIndex" />
  </div>
</template>

<script>
import Chart from "@/components/BinaryOptionChart.vue";
import Controls from "@/components/GameControls.vue";
import OptionBetsList from "@/components/OptionBetsList.vue";
import rates_data from "@/assets/rates-data.js";
import c_bet from "@/consts/bet.js";
import c_storage from "@/consts/storage.js";
import { openDB } from 'idb';

export default {
  components: { Chart, Controls, OptionBetsList },
  data() {
    return {
      balance: null,
      displayedRates: [],
      bets: [],
      currentIndex: 0,
      db: null,
      ratesToDisplay: 100,
      ratesLength: rates_data.length,
    };
  },
  async created() {
    this.db = await openDB(c_storage.DBName, 1, {
      upgrade(db) {
        db.createObjectStore(c_storage.CollectionName, { keyPath: 'id' });
      },
    });

    this.balance = (await this.db.get(c_storage.CollectionName, c_storage.BalanceKey))?.value ?? 1000;

    this.currentIndex = (await this.db.get(c_storage.CollectionName, c_storage.CurrentIndexKey))?.value
      ?? Math.floor(Math.random() * this.ratesLength % this.ratesToDisplay) + 100;

    const storedBets = await this.db.get(c_storage.CollectionName, c_storage.BetsKey);
    this.bets = storedBets ? JSON.parse(storedBets.value) : [];

    await this.startSimulation();
  },
  methods: {
    async startSimulation() {

      this.displayedRates = rates_data
        .slice(this.currentIndex, this.currentIndex + this.ratesToDisplay);

      setInterval(async () => {
        if (this.currentIndex < rates_data.length - 1) {
          this.currentIndex++;
          this.displayedRates = rates_data.slice(this.currentIndex - this.ratesToDisplay, this.currentIndex);
          await this.db.put(c_storage.CollectionName, { id: c_storage.CurrentIndexKey, value: this.currentIndex });

          // Process bets
          let expiredBets = this.bets.filter(bet =>
            bet.status === c_bet.ActiveStatus &&
            bet.startIndex + bet.timeframe <= this.currentIndex
          );

          expiredBets.forEach(async bet => {
            const isSuccessful =
              (bet.type === c_bet.CallType && rates_data[this.currentIndex] > bet.rate) ||
              (bet.type === c_bet.PutType && rates_data[this.currentIndex] < bet.rate);

            if (isSuccessful) {
              bet.status = c_bet.WonStatus;

              await this.updateBalance(this.balance + bet.amount * 2);
            }
            else {
              bet.status = c_bet.LostStatus;
            }
          });

          if (expiredBets.length) {
            await this.updateBets(this.bets);
          }
        }
      }, 1000);
    },
    async handleAction(action) {
      if (this.balance < action.bet) {
        alert('Insufficient balance');
        return;
      }

      const bet = {
        type: action.type,
        amount: action.bet,
        timeframe: +action.timeframe,
        startIndex: this.currentIndex,
        rate: recalculateRate(rates_data[this.currentIndex], action.type, action.timeframe),
        status: c_bet.ActiveStatus
      };

      await this.updateBalance(this.balance - action.bet);

      this.bets.unshift(bet);
      await this.updateBets(this.bets);
    },
    async resetBalance() {
      await this.updateBalance(1000);
      await this.updateBets([]);
    },

    async updateBalance(balance) {
      this.balance = balance;
      await this.db.put(c_storage.CollectionName, { id: c_storage.BalanceKey, value: this.balance });
    },
    async updateBets(bets) {
      this.bets = bets;
      await this.db.put(c_storage.CollectionName, { id: c_storage.BetsKey, value: JSON.stringify(bets) });
    }
  }
};

const recalculateRate = (rate, type, timeframe) => {
  const baseRatePercentage = 0.004;
  const adjustedRatePercentage = baseRatePercentage * (1 + (3 / timeframe));

  const newRate = rate * (type === c_bet.CallType ? (1 + (adjustedRatePercentage / 100)) : (1 - (adjustedRatePercentage / 100)));
  return parseFloat(newRate.toFixed(5));
};
</script>


<style scoped>
.controls-container {
  display: flex;
  justify-content: center;
  align-items: center;
}

.controls {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.buttons {
  display: flex;
  flex-direction: row;
  justify-content: center;
  gap: 300px;
}

label {
  margin-right: 10px;
}

button {
  margin-top: 10px;
}
</style>