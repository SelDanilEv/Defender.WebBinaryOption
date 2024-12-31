<template>
    <div class="bet-list">
        <h3>Bets</h3>
        <table>
            <thead>
                <tr>
                    <th>Type</th>
                    <th>Amount</th>
                    <th>Timeframe</th>
                    <th>Rate</th>
                    <th>Result</th>
                </tr>
            </thead>
            <tbody>
                <tr v-for="bet in bets" :key="bet.startIndex">
                    <td>{{ bet.type }}</td>
                    <td>{{ bet.amount }}</td>
                    <td>{{ bet.timeframe }}s</td>
                    <td>{{ bet.rate }}</td>
                    <td>
                        <span v-if="bet.startIndex + bet.timeframe <= currentIndex">{{ bet.status }}</span>
                        <span v-else>{{ bet.startIndex + bet.timeframe - currentIndex }}s</span>
                    </td>
                </tr>
            </tbody>
        </table>
    </div>
</template>

<script>
import c_props from "@/consts/props.js";

export default {
    props: [c_props.Bets, c_props.CurrentIndex],
};
</script>

<style scoped>
.bet-list {
    margin-top: 20px;
    color: #ffffff;
}

.bet-list table {
    width: 100%;
    border-collapse: collapse;
}

.bet-list th,
.bet-list td {
    border: 1px solid #444444;
    padding: 10px;
    text-align: left;
}

.bet-list th {
    background-color: #333333;
}

.bet-list td {
    background-color: #222222;
}
</style>