<template>
  <q-page class="q-pa-md flex flex-center" style="color: white">
    <div class="text-center">
      <h1 class="text-h3 q-mb-md q-px-sm">Космос в цифрах и графиках</h1>
      <span class="text-body1 q-mt-md q-px-sm">Следующий запуск:</span>
      <h3 class="text-h5 q-mt-md q-px-sm" style="text-color: #858585">
        {{ nextLaunchName }} ({{ nextLaunchProvider }})
      </h3>

      <!-- Таймер -->
      <q-card
        flat
        bordered
        class="q-my-md text-center"
        style="background: rgba(255, 255, 255, 0)"
      >
        <q-card-section class="row justify-center">
          <div
            v-for="(unit, index) in timerUnits"
            :key="index"
            class="col-auto q-pa-md"
            style="
              background: rgba(255, 255, 255, 0.1);
              margin-left: 25px;
              padding: 30px;
            "
          >
            <div class="text-h2 text-yellow">{{ unit.value }}</div>
            <div class="text-h6 text-yellow">{{ unit.label }}</div>
          </div>
        </q-card-section>
      </q-card>

      <!-- Таблица с предстоящими запусками -->
      <!-- <q-card
        flat
        bordered
        class="q-my-md"
        style="background: rgba(255, 255, 255, 0.1)"
      >
        <q-card-section>
          <h3 class="text-body1">Предстоящие запуски</h3>
          <q-table
            flat
            dense
            :rows="launches"
            :columns="columns"
            row-key="id"
            class="q-my-md"
            style="background: rgba(255, 255, 255, 1); color: black"
          >
            <template v-slot:body-cell-description="props">
              <q-td>{{ props.row.launch_description }}</q-td>
            </template>
            <template v-slot:body-cell-t0="props">
              <q-td>{{ formatDateTime(props.row.t0) }}</q-td>
            </template>
          </q-table>
        </q-card-section>
      </q-card> -->
      <q-card
        flat
        bordered
        class="q-my-md"
        style="background: rgba(255, 255, 255, 0.1)"
      >
        <q-card-section>
          <div class="launch-list">
            <q-card
              flat
              v-for="(launch, index) in launches"
              :key="index"
              class="q-my-sm q-px-sm launch-card"
              :class="{ 'highlight-next': isNextLaunch(launch) }"
            >
              <q-card-section>
                <div class="row">
                  <!-- Дата и время -->
                  <div class="col-auto text-left q-pa-md">
                    <div class="text-bold text-h6">
                      {{ formatDate(launch.t0) }}
                    </div>
                    <div class="text-caption">{{ formatTime(launch.t0) }}</div>
                  </div>

                  <!-- Основная информация -->
                  <div class="col text-left">
                    <div class="text-bold text-h5">
                      {{ launch.vehicle?.name || 'Неизвестно' }}
                    </div>
                    <div class="text-caption">
                      {{ launch.provider?.name || 'Неизвестный провайдер' }}
                    </div>
                    <div class="text-caption text-italic">
                      {{ launch.pad?.location?.name || 'Неизвестная площадка' }}
                    </div>
                    <div class="text-caption truncate-text">
                      {{ launch.launch_description || 'Описание отсутствует' }}
                    </div>
                  </div>

                  <!-- Дополнительная информация -->
                  <div class="col-auto text-right">
                    <!-- Теги -->
                    <div class="q-mt-sm">
                      <q-chip
                        v-for="(tag, idx) in launch.tags"
                        :key="idx"
                        dense
                        outline
                        color="blue-5"
                        text-color="blue-10"
                        class="q-mr-xs"
                      >
                        {{ tag.text }}
                      </q-chip>
                    </div>
                  </div>
                </div>
              </q-card-section>
            </q-card>
          </div>
        </q-card-section>
      </q-card>
    </div>
  </q-page>
</template>

<script>
import axios from 'axios';

export default {
  data() {
    return {
      // Таймер
      timerUnits: [
        { label: 'Дней', value: '00' },
        { label: 'Часов', value: '00' },
        { label: 'Минут', value: '00' },
        { label: 'Секунд', value: '00' },
      ],

      // Данные для таблицы
      launches: [],
      columns: [
        {
          name: 'name',
          label: 'Название миссии',
          align: 'left',
          field: 'name',
        },
        {
          name: 'provider',
          label: 'Провайдер',
          align: 'left',
          field: (row) => row.provider?.name || 'Не указано',
        },
        {
          name: 'vehicle',
          label: 'Ракета',
          align: 'left',
          field: (row) => row.vehicle?.name || 'Не указано',
        },
        {
          name: 'pad',
          label: 'Площадка',
          align: 'left',
          field: (row) => row.pad?.location?.name || 'Не указано',
        },
        {
          name: 'launch_description',
          label: 'Описание',
          align: 'left',
          field: 'launch_description',
        },
        { name: 't0', label: 'Запланирован', align: 'center', field: 't0' },
      ],
      nextLaunchDate: null,
      nextLaunchName: null,
      nextLaunchProvider: null,
    };
  },
  methods: {
    async fetchLaunchData() {
      try {
        const response = await axios.get('https://ingeni.space/api/');
        if (response.data && response.data.result) {
          this.launches = response.data.result;
          this.findNextLaunch();
        }
      } catch (error) {
        console.error('Ошибка загрузки данных о запусках:', error);
      }
    },
    findNextLaunch() {
      const now = new Date();
      const futureLaunches = this.launches.filter(
        (launch) => new Date(launch.t0) > now
      );
      if (futureLaunches.length > 0) {
        const nextLaunch = futureLaunches.reduce((earliest, current) =>
          new Date(current.t0) < new Date(earliest.t0) ? current : earliest
        );
        this.nextLaunchDate = new Date(nextLaunch.t0);
        this.nextLaunchName = nextLaunch.name;
        this.nextLaunchProvider = nextLaunch.provider.name;
      } else {
        this.nextLaunchDate = null;
        this.nextLaunchName = 'Нет информации о запланированных запусках';
        this.nextLaunchProvider = 'Неизвестный провайдер';
      }
    },
    updateTimer() {
      if (this.nextLaunchDate) {
        const now = new Date();
        const diff = Math.max(0, this.nextLaunchDate - now); // Временная разница в миллисекундах

        const days = Math.floor(diff / (1000 * 60 * 60 * 24));
        const hours = Math.floor(
          (diff % (1000 * 60 * 60 * 24)) / (1000 * 60 * 60)
        );
        const minutes = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
        const seconds = Math.floor((diff % (1000 * 60)) / 1000);

        this.timerUnits = [
          { label: 'Дней', value: String(days).padStart(2, '0') },
          { label: 'Часов', value: String(hours).padStart(2, '0') },
          { label: 'Минут', value: String(minutes).padStart(2, '0') },
          { label: 'Секунд', value: String(seconds).padStart(2, '0') },
        ];
      }
    },
    formatDateTime(dateTime) {
      const options = {
        year: 'numeric',
        month: 'numeric',
        day: 'numeric',
        hour: '2-digit',
        minute: '2-digit',
      };
      return new Date(dateTime).toLocaleString('ru-RU', options);
    },
    formatDate(dateTime) {
      const options = { month: 'short', day: 'numeric', year: 'numeric' };
      return new Date(dateTime).toLocaleDateString('en-US', options);
    },
    formatTime(dateTime) {
      const options = { hour: '2-digit', minute: '2-digit', hour12: true };
      return new Date(dateTime).toLocaleTimeString('en-US', options);
    },
    isNextLaunch(launch) {
      const isMatch =
        this.nextLaunchDate &&
        new Date(launch.t0).getTime() === this.nextLaunchDate.getTime();
      console.log('Проверка запуска:', launch.name, 'Результат:', isMatch);
      return isMatch;
    },
  },
  mounted() {
    this.fetchLaunchData();
    this.timer = setInterval(this.updateTimer, 1000);
  },
  beforeUnmount() {
    clearInterval(this.timer);
  },
};
</script>

<style>
body {
  background-image: url('https://ingeni.space/images/img2.jpg');
  background-size: cover;
  background-position: center;
}

.launch-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.launch-card {
  background: rgba(255, 255, 255, 0.05);
  border: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 8px;
  overflow: hidden;
}

.launch-card:hover {
  background: rgba(255, 255, 255, 0.1);
  transition: background 0.3s;
}

.truncate-text {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  max-width: 100%;
}

.highlight-next {
  background: rgba(255, 235, 59, 0.2); /* Светло-желый  фон */
  border: 2px solid rgba(255, 235, 59, 0.5); /* Светло-желтая граница */
  transition: background 0.3s, border 0.3s;
}
</style>
