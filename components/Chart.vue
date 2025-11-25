<template>
  <div :id="chartId" class="chart-container"></div>
</template>

<script setup>
import { onMounted, onBeforeUnmount, watch } from 'vue'
import Highcharts from 'highcharts'

const props = defineProps({
  chartId: {
    type: String,
    required: true
  },
  data: {
    type: Object,
    required: true
  }
})

let chart = null

// Функция для преобразования строки с пробелами в число
const parseNumber = (value) => {
  if (!value) return 0
  // Удаляем все пробелы и преобразуем в число
  return Number(value.toString().replace(/\s/g, '')) || 0
}

onMounted(() => {
  chart = Highcharts.chart(props.chartId, {
    chart: {
      type: 'line',
      height: 500
    },
    title: {
      text: props.data.indicator || ''
    },
    xAxis: {
      categories: ['Текущий день', 'Вчера', 'Этот день недели']
    },
    yAxis: {
      title: {
        text: 'Значение'
      },
      gridLineWidth: 0
    },
    series: [{
      name: props.data.indicator,
      color: '#34495e',
      data: [
        parseNumber(props.data.currentDay),
        parseNumber(props.data.yesterday),
        parseNumber(props.data.thisWeek)
      ]
    }],
    credits: {
      enabled: false
    }
  })
})

onBeforeUnmount(() => {
  if (chart) {
    chart.destroy()
  }
})

watch(() => props.data, () => {
  if (chart) {
    chart.series[0].update({
      name: props.data.indicator,
      data: [
        Number(props.data.currentDay),
        Number(props.data.yesterday),
        Number(props.data.thisWeek)
      ]
    })
  }
}, { deep: true })
</script>

<style scoped>
.chart-container {
  width: 100%;
  margin-top: 10px;
}
</style>
