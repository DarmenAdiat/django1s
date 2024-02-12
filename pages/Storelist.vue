<template>
  <v-container>
    <v-row>
      <v-col cols='10' style='margin: auto;'>
        <v-data-table
          :headers="headers"
          :items="storages"
          :items-per-page='-1'
          :search="search"
          class="elevation-1"
          style='cursor: pointer'
          hide-default-footer
          :header-props="headerProps"
          :loading='loadTable'
          loading-text='Загрузка данных'
          @click:row="onClickRow"
        >
          <template #top>
            <h3 style='margin: 10px;'>Storages</h3>
            <v-row style='margin: auto'>
              <v-col cols='12' sm='12' lg='6' md='6'>
              <v-text-field
                v-model="search"
                style='max-width:600px;margin: 4px auto auto;'
                append-icon="mdi-magnify"
                label="Поиск"
                single-line
                hide-details
              ></v-text-field>
              </v-col>
              <v-col cols='12' sm='12' lg='6' md='6'>
              <v-form ref='form' style='width:100%;min-width: 250px;max-width: 600px;margin: auto;'>
              <v-dialog
                ref="dialog"
                v-model="modal"
                :return-value.sync="date"
                persistent
                width="290px"
              >
                <template #activator="{ on, attrs }">
                  <v-text-field
                    v-model="date2"
                    label="Выберите дату списания"
                    prepend-icon="mdi-calendar"
                    :rules="dateRules"
                    required
                    readonly
                    v-bind="attrs"
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="date"
                  :first-day-of-week="1"
                  :allowed-dates="disablePastDates"
                  locale="ru-ru"
                  color="primary"
                  scrollable
                >
                  <v-spacer></v-spacer>
                  <v-btn
                    text
                    color="primary"
                    @click="modal = false"
                  >
                    Cancel
                  </v-btn>
                  <v-btn
                    text
                    color="primary"
                    @click="$refs.dialog.save(date)"
                  >
                    OK
                  </v-btn>
                </v-date-picker>
              </v-dialog>
              </v-form>
              </v-col>
            </v-row>
          </template>
          <template #no-data>
            <v-alert :value="true" color="error" icon="warning">
              На Ваше имя нет зарегистированных помещений
            </v-alert>
            <v-btn
              color="primary"
              style='margin-bottom: 10px;'
              @click="initialize"
            >
              Обновить
            </v-btn>
          </template>
          <template  #no-results>
            <v-alert :value="true" color="error" icon="warning">
              По вашему поиску "{{ search }}" не найдено результатов.
            </v-alert>
          </template>

        </v-data-table>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import moment from 'moment'

export default {
  name: 'Storelist',
  middleware: 'auth',
  data: () => ({
    date: moment(new Date()).format('yyyy-MM-DD'),
    date2: '',
    modal: false,
    search: '', // поисковик
    loadTable: true, // загрузчик таблицы

    dateRules: [
      v => !!v || 'Необходимо выбрать дату',
    ],

    headerProps: {
      sortByText: "Сортировать таблицу",
    },
    headers: [
      { text: 'Location', value: 'location', },
      { text: 'Менеджер проекта', value: 'manager', },
      { text: 'Склад', value: 'storage', },
      { text: 'Помещение', value: 'placement', },
    ],
    storages: [],
  }),
  head: {
    title: 'MAWS-Мои помещения',
  },
  computed: {
    user () {
      return this.$auth.user
    },
  },
  created () {
    this.initialize()
  },

  methods: {
    async initialize() {
      try {
        this.storages = await this.$axios.$get('http://nls-1c/HttpService/hs/Web1C/V2', {
          params: {
            iin_num: this.user.id_num, // отправляем в гет запросе ИИН авторизованного пользователя
            // iin_num: '880315400615', // отправляем в гет запросе ИИН авторизованного пользователя
          }
        })
        this.loadTable = false
        if (this.storages === 'По этому ИИН нет помещении') {
          this.storages = []
          console.log('wtf' + this.storages)
        }
        return this.storages // возвращаем полученное значение функции
      } catch (e) {
          console.log(e)
      }
    },

    disablePastDates(val){
      const createdDate = moment(new Date()).format('yyyy-MM-DD')
      const lastDate = moment(new Date()).subtract(31, 'day').format('yyyy-MM-DD')
      this.date2 = moment(this.date).format('DD.MM.yyyy')
      return val <= createdDate && val > lastDate
    },

    onClickRow(item){
      // console.log(item)
      if (this.date2 === '') {
        this.$refs.form.validate();
      } else {
        this.$router.push({
          path: 'Maws',
          query: {
            placement: item.placement,
            storage: item.storage,
            location: item.location,
            manager: item.manager,
            date: this.date }
        });
      }

    },

    pass () {
      alert('almost right here dude!')
    },
  },

}
</script>

<style scoped>

</style>
