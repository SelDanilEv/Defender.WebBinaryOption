<template>
  <div style="height: 400px; width: 100%;">
    <Line :data="chartData" :options="chartOptions" />
  </div>
</template>

<script>
import { Line } from "vue-chartjs";
import {
  Chart as ChartJS,
  Title,
  Tooltip,
  Legend,
  LineElement,
  PointElement,
  CategoryScale,
  LinearScale,
} from "chart.js";
import c_bet from "@/consts/bet.js";
import c_props from "@/consts/props.js";
import annotationPlugin from 'chartjs-plugin-annotation';

ChartJS.register(
  Title,
  Tooltip,
  Legend,
  LineElement,
  PointElement,
  CategoryScale,
  LinearScale,
  annotationPlugin
);

export default {
  components: { Line },
  props: [c_props.Data, c_props.Bets],
  computed: {
    chartData() {
      if (!this.data || this.data.length === 0) {
        return {
          labels: [],
          datasets: [],
        };
      }

      const now = new Date();

      const chartData = {
        labels: this.data.map((_, index) => {
          const date = new Date(now);
          date.setSeconds(now.getSeconds() - (this.data.length - 1 - index));
          return date.toLocaleTimeString();
        }),
        datasets: [
          {
            label: "EUR/USD",
            data: this.data,
            borderColor: "rgba(75, 192, 192, 1)",
            borderWidth: 2,
            fill: false,
          },
        ],
      };
      return chartData;
    },
    chartOptions() {
      const annotations = this.bets.filter(bet => bet.status === c_bet.ActiveStatus).map(bet => ({
        type: 'line',
        mode: 'horizontal',
        scaleID: 'y',
        value: bet.rate,
        borderColor: bet.type === c_bet.CallType ? 'green' : 'red',
        borderWidth: 2,
      }));

      return {
        responsive: true,
        maintainAspectRatio: false,
        animation: false,
        plugins: {
          legend: { display: false },
          tooltip: { enabled: false },
          annotation: {
            annotations: annotations
          }
        },
      };
    }
  }
};
</script>