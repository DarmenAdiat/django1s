<template>
  <v-container>
    <v-row justify="center" align="center">
      <v-col cols="12" style='margin-top:20px;'>
        <h1>Все документы</h1>
        <div>
          <v-data-table
            :headers="headers"
            :items="ordersWithIndex"
            item-key="name"
            class="elevation-1"
            style='cursor: pointer'
            :search="search"
            :footer-props="footerProps"
            :header-props="headerProps"
          >
            <template #[`item.doc_status`]="{ item }">
              <span
                v-if='item.doc_status==="Черновик"'
                style='color: black'
              >
                {{ item.doc_status }}
              </span>
              <span
                v-if='item.doc_status==="Приказ выпущен"'
                style='color: #4ECDC4'
              >
                {{ item.doc_status }}
              </span>
              <span
                v-if='item.doc_status==="Заявка одобрена"'
                style='color: #ff8f00'
              >
                {{ item.doc_status }}
              </span>
              <span
                v-if='item.doc_status==="На согласовании"'
                style='color: #ff8f00'
              >
                {{ item.doc_status }}
              </span>
              <span
                v-if='item.doc_status==="Заявка отклонена"'
                style='color: #FF6B6B'
              >
                {{ item.doc_status }}
              </span>
              <span
                v-if='item.doc_status==="Списано"'
                style='color: #4ECDC4'
              >
                {{item.doc_status}}
              </span>
              <span
                v-if='item.doc_status==="Отправлено, ожидает списания"'
                style='color: #FFC107'
              >{{ item.doc_status }}</span>
              <span
                v-if='item.doc_status==="Частично исполнено"'
                style='color: #ff8f00'
              >{{ item.doc_status }}</span>
              <span
                v-if='item.doc_status==="Нет статуса"'
                style='color: black'
              >{{ item.doc_status }}
              </span>
            </template>
            <template #top>
              <v-text-field
                v-model="search"
                label="Поиск"
                class="mx-4"
                append-icon="mdi-magnify"
                single-line
                hide-details
              ></v-text-field>
            </template>
            <template #no-data>
                У Вас еще нет оправленных документов
            </template>
            <template  #no-results>
              <v-alert :value="true" color="error" icon="warning">
                По вашему поиску "{{ search }}" не найдено результатов.
              </v-alert>
            </template>
          </v-data-table>
        </div>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import moment from 'moment';

export default {
  name: 'Loglist',
  middleware: 'auth',

  data() {
    return {
      search: '',
      loglist: [],
      options: {},
      headerProps: {
        sortByText: "Сортировать список"
      },
      footerProps: {
        'disable-items-per-page': true,
        'items-per-page-text': 'Документов на странице'
      },
    }
  },
  head: {
    title: 'Список документов',
  },
  computed: {
      headers() {
        return [
          {text: '#', value: 'index'},
          {text: 'Наименование документа', value: 'doc_name'},
          {text: 'Дата создания', value: 'doc_date', format: 'YYYY-MM-DD HH:mm:ss'},
          {text: 'Статус', value: 'doc_status'},
        ]
      },
      ordersWithIndex() {
        return this.loglist.map((items, index) => ({...items, index: index + 1 }))
      },
      user() {
        return this.$auth.user
      },
  },

  created() {
    this.initialize()
  },

  methods: {
    async initialize() {
      const response = await this.$axios.$get(`/api/v1/loglist/`, {
        params: {
          id_user: this.user.id,
        },
      })

      for (let i = 0; i < response.length; i++) {
        response[i].doc_status = this.changeStatus(response[i].doc_name, response[i].doc_status)
        response[i].doc_date = moment(response[i].doc_date).format('HH:mm - DD/MM/yyyy');
      }

      this.loglist = response
      return this.loglist
    },


    changeStatus(name, status){
      let newStatus = ''
      if (name === 'Заявка на списание') {
        if (status === '1') {
          newStatus = 'Принято'
        }
        else if (status === '2') {
          newStatus = 'Отправлено, ожидает списания'
        }
        else if (status === '3') {
          newStatus = 'Списано'
        }
        else if (status === '4') {
          newStatus = 'Отклонено'
        }
        else if (status === '5') {
          newStatus = 'Частично исполнено'
        }
        else if (status === '6') {
          newStatus = 'Черновик'
        }
        else {
          newStatus = 'Нет статуса'
        }
      }
      if (name !== 'Заявка на списание') {
        if (status === '1') {
          newStatus = 'Заявка одобрена'
        }
        else if (status === '2') {
          newStatus = 'На согласовании'
        }
        else if (status === '3') {
          newStatus = 'Приказ выпущен'
        }
        else if (status === '4') {
          newStatus = 'Заявка отклонена'
        }
        else if (status === '5') {
          newStatus = 'Ошибка'
        }
        else if (status === '6') {
          newStatus = 'Черновик'
        }
        else {
          newStatus = 'Нет статуса'
        }
      }

      return newStatus
    },

  },

}
</script>
