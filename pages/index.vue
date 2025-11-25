<template>

  <section class="section">
    <div class="container">
      <h1>Table to Nuxt 3</h1>
      <p v-if="lastUpdateDate" class="last-update">
        Последнее обновление: {{ lastUpdateDate }}
      </p>
      <div class="table__container">
        <table class="table">
          <thead class="table__header">
            <tr>
              <th class="table__headerCell" v-for="(title, index) in tableParams.title" :key="index">
                {{ title }}
              </th>
            </tr>
          </thead>
          <tbody class="table__body">
            <template v-for="(item, index) in tableParams.data" :key="index">
              <tr @click="toggleChart(index)" class="table__bodyRow">
                <td class="table__bodyCell indicator" :data-label="tableParams.title[0]">{{ item.indicator }}</td>
                <td class="table__bodyCell table__bodyCell--yesterday" :data-label="tableParams.title[1]">{{ item.currentDay }}</td>
                <td class="table__bodyCell table__bodyCell--currentDay" :class="getPercentClass(item)" :data-label="tableParams.title[2]">
                  <div class="currentDay__container">
                    <div class="currentDay__value">{{ item.yesterday }}</div>
                    <div class="currentDay__percent" v-if="getChangePercent(item) !== null">
                      {{ formatPercent(getChangePercent(item)) }}
                    </div>
                  </div>
                </td>
                <td class="table__bodyCell table__bodyCell--thisWeek" :class="getThisWeekClass(item)" :data-label="tableParams.title[3]">
                  {{ item.thisWeek }}
                </td>
              </tr>
              <tr v-if="openCharts[index]" class="table__chartRow">
                <td colspan="4" class="table__chartCell">
                  <Chart :chart-id="`chart-${index}`" :data="item" />
                </td>
              </tr>
            </template>
          </tbody>
        </table>
      </div>
    </div>
  </section>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const openCharts = ref({})
const lastUpdateDate = ref(null)

const tableParams = ref({
  title: ['Показатель', 'Текущий день', 'Вчера', 'Этот день недели'],
  data: []
})

//текущая дата
const getCurrentDate = () => {
  return new Date().toISOString().split('T')[0]
}

const shouldUpdateData = () => {
  const today = getCurrentDate()
  const lastUpdate = localStorage.getItem('lastDataUpdate')

  //обновление даты
  return !lastUpdate || lastUpdate !== today
}

// Функция для преобразования строки с пробелами в число
const parseNumber = (value) => {
  if (!value) return 0
  return Number(value.toString().replace(/\s/g, '')) || 0
}

// Вычисление процента изменения
const getChangePercent = (item) => {
  const current = parseNumber(item.currentDay)
  const yesterday = parseNumber(item.yesterday)

  // Если вчерашнее значение равно 0, не можем вычислить процент
  if (yesterday === 0) {
    return current > 0 ? 100 : (current < 0 ? -100 : null)
  }

  return ((yesterday - current) / current) * 100
}

// убираем знаки после запятой
const formatPercent = (percent) => {
  if (percent === null) return ''
  return `${percent.toFixed(0)}%`
}

// Получение класса для ячейки "Вчера" на основе процента
const getPercentClass = (item) => {
  const percent = getChangePercent(item)
  if (percent === null) return ''

  if (percent > 0) {
    return 'table__bodyCell--positive'
  } else if (percent < 0) {
    return 'table__bodyCell--negative'
  } else {
    return ''
  }
}

const getThisWeekClass = (item) => {
  const thisWeek = parseNumber(item.thisWeek)
  const yesterday = parseNumber(item.yesterday)

  if (thisWeek > yesterday) {
    return 'table__bodyCell--negative'
  } else if (thisWeek < yesterday) {
    return 'table__bodyCell--positive'
  } else {
    return ''
  }
}

const fetchData = async () => {
  //запрос к API
  //const response = await $fetch('/api/indicators')
  //return response.data

  //временные данные
  return [
    {
      indicator: 'Выручка, руб',
      yesterday: '500 521',
      currentDay: '480 521',
      thisWeek: '4 805 121',
    },
    {
      indicator: 'Наличные',
      yesterday: '300 000',
      currentDay: '300 000',
      thisWeek: '300 000',
    },
    {
      indicator: 'Безналичный расчет',
      yesterday: '100 000',
      currentDay: '100 000',
      thisWeek: '100 000',
    },
    {
      indicator: 'Кредитные карты',
      yesterday: '100 521',
      currentDay: '100 521',
      thisWeek: '100 521',
    },
    {
      indicator: 'Средний чек, руб',
      yesterday: '1 300',
      currentDay: '900',
      thisWeek: '900',
    },
    {
      indicator: 'Средний гость, руб',
      yesterday: '1 200',
      currentDay: '800',
      thisWeek: '800',
    },
    {
      indicator: 'Удаления из чека (после оплаты), руб',
      yesterday: '1 000',
      currentDay: '1 100',
      thisWeek: '900',
    },
    {
      indicator: 'Удаления из чека (до оплаты), руб',
      yesterday: '1 300',
      currentDay: '1 300',
      thisWeek: '900',
    },
    {
      indicator: 'Количество чеков',
      yesterday: '36',
      currentDay: '34',
      thisWeek: '34',
    },
    {
      indicator: 'Количество гостей',
      yesterday: '34',
      currentDay: '36',
      thisWeek: '32',
    },
  ]
}

//обновление данных
const updateData = async () => {
  try {
    const data = await fetchData()
    tableParams.value.data = data

    const today = getCurrentDate()
    const updateDateTime = new Date().toLocaleString('ru-RU')

    localStorage.setItem('lastDataUpdate', today)
    localStorage.setItem('tableData', JSON.stringify(data))
    localStorage.setItem('lastUpdateDate', updateDateTime)
  } catch (error) {
    console.error('Ошибка при загрузке данных:', error)
  }
}

//показ графика
const toggleChart = (index) => {
  openCharts.value[index] = !openCharts.value[index]
}

//обновление данных при загрузке
onMounted(async () => {
  if (shouldUpdateData()) {
    await updateData()
  } else {
    const savedData = localStorage.getItem('tableData')
    if (savedData) {
      try {
        tableParams.value.data = JSON.parse(savedData)
        const savedDate = localStorage.getItem('lastUpdateDate')
        lastUpdateDate.value = savedDate || new Date().toLocaleString('ru-RU')
      } catch (error) {
        console.error('Ошибка при загрузке данных из localStorage:', error)
        await updateData()
      }
    } else {
      await updateData()
    }
    lastUpdateDate.value = localStorage.getItem('lastUpdateDate') || 'Неизвестно'
  }

  //проеряем смену даты при возвращении на вкладку
  document.addEventListener('visibilitychange', () => {
    if (!document.hidden && shouldUpdateData()) {
      console.log('обновление данных')
      updateData()
    }
  })
})

</script>

<style lang="scss" scoped>
.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 2rem;
  text-align: center;
  justify-content: center;

  @include phones {
    padding: 1rem;
  }

  h1 {
    color: #42b983;
    font-size: 2.5rem;
    margin-bottom: 1rem;

    @include phones {
      font-size: 1.5rem;
    }
  }

  p {
    color: #666;
    font-size: 1.2rem;

    @include phones {
      font-size: 1rem;
    }
  }
}

.section {
  width: 100%;
  align-items: center;
  overflow-x: hidden;
}

.table__container {
  display: flex;
  justify-content: center;
  width: 100%;
  overflow-x: auto;
  -webkit-overflow-scrolling: touch;

  @include phones {
    justify-content: flex-start;
  }
}

.table {
  margin-top: 2rem;
  width: 100%;
  min-width: 600px;
  border-collapse: collapse;

  @include phones {
    min-width: 100%;
    display: block;
  }

  &__header {
    @include phones {
      display: none;
    }
  }

  &__headerCell {
    font-weight: 600;
    color: #7c7f81;
    background-color: #f2f2f2;
    text-align: center;
    padding: 12px;
    white-space: nowrap;

    &:nth-child(2) {
      background-color: #edf8ff;
    }

    @include phones {
      display: none;
    }
  }

  &__body {
    @include phones {
      display: block;
    }
  }

  &__bodyRow {
    cursor: pointer;
    transition: background-color 0.2s;

    &:hover {
      background-color: #f8f9fa;
    }

    @include phones {
      display: block;
      margin-bottom: 15px;
      border: 1px solid #ddd;
      border-radius: 8px;
      background-color: #fff;
      padding: 12px;
    }
  }

  &__bodyCell {
    font-weight: 600;
    text-align: right;
    color: #7c7f81;
    background-color: #F5F5F5;
    align-items: center;
    padding: 12px;
    white-space: nowrap;

    @include phones {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 8px 12px;
      border: none;
      background-color: transparent;
      text-align: left;

      &::before {
        content: attr(data-label);
        font-weight: 600;
        color: #7c7f81;
        margin-right: 10px;
        flex-shrink: 0;
      }
    }

    &:first-child {
      text-align: left;

      @include phones {
        font-weight: 700;
        font-size: 1.1em;
        padding-bottom: 8px;
        margin-bottom: 8px;
        border-bottom: 2px solid #f2f2f2;

        &::before {
          content: '';
        }
      }
    }

    &--yesterday {
      background-color: #edf8ff;
    }

    .currentDay__container {
      display: flex;
      justify-content: right;
      align-items: center;

      @include phones {
        flex-wrap: wrap;
      }
    }

    &--currentDay {
      .currentDay__value {
        margin: 0 10px;
      }

      .currentDay__percent {
        font-size: 0.85em;
        font-weight: bold;
        color: #288e38;
      }
    }

    &--thisWeek {
      background-color: #F5F5F5;
    }

    &.table__bodyCell--positive {
      background-color: #ecf7e7;
    }

    &.table__bodyCell--negative {
      background-color: #fee6e6;

      .currentDay__percent {
        color: #ff3d42;
      }
    }
  }

}

// Стили для строки с графиком
.table__chartRow {
  @include phones {
    display: block;
    width: 100%;
    margin-bottom: 15px;
  }

  .table__chartCell {
    @include phones {
      display: block;
      width: 100%;
      padding: 10px;
    }
  }
}

.last-update {
  text-align: left;
  color: #666;
  font-size: 0.9rem;
  margin-top: 1rem;

  @include phones {
    font-size: 0.8rem;
    text-align: center;
  }
}
</style>
