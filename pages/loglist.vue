<template>
  <v-container>
    <v-row justify="center" align="center">
      <v-col cols="12" sm="12" md="10" lg="8">
        <h1>Список</h1>
        <div>
          <v-data-table
            :headers="headers"
            :items="ordersWithIndex"
            item-key="name"
            class="elevation-1"
            :search="search"
            :footer-props="footerProps"
          >
            <template v-slot:top>
              <v-text-field
                v-model="search"
                label="Поиск"
                class="mx-4"
              ></v-text-field>
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
  middleware: 'auth',

  async asyncData(context) {
    const response = await context.app.$axios.$get(`/api/v1/loglist`)

    for (let i = 0; i < response.length; i++) {

      if (response[i].doc_status === '0'){
        response[i].doc_status = 'Отправлено'
      }
      else if (response[i].doc_status === '1') {
        response[i].doc_status = 'Принято'
      }
      else if (response[i].doc_status === '2') {
        response[i].doc_status = 'На согласовании'
      }
      else if (response[i].doc_status === '3') {
        response[i].doc_status = 'Согласовано'
      }
      else if (response[i].doc_status === '4') {
        response[i].doc_status = 'Отклонено'
      }
      else if (response[i].doc_status === '5') {
        response[i].doc_status = 'Ошибка'
      }

      response[i].doc_date = moment(response[i].doc_date).format('HH:mm:ss - DD/MM/yyyy');
      console.log(response[i].doc_date)
    }
    console.log(response.length)

    return { loglist: response }
  },

  data() {
    return {
      search: '',
      options: {},
      footerProps: {
        'disable-items-per-page': true,
        'items-per-page-text': 'Документов на странице'
      },
    }
  },

  computed: {
      headers() {
        return [
          {text: '#', value: 'index'},
          {text: 'Имя', value: 'first_name'},
          {text: 'Фамилия', value: 'last_name'},
          {text: 'Организация', value: 'full_name'},
          {text: 'Наименование документа', value: 'doc_name'},
          // {text: 'Дата создания', value: 'doc_date', dataType: 'string'},
          {text: 'Дата создания', value: 'doc_date', format: 'YYYY-MM-DD HH:mm:ss'},
          {text: 'Статус', value: 'doc_status'},
        ]
      },
      ordersWithIndex() {
        return this.loglist.map((items, index) => ({...items, index: index + 1 }))
      },

  },
  methods: {


  },

}
</script>
