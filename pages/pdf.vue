<template>
  <v-container>
    <v-row id="downloadM29" style='margin: auto;padding: 0;width:1200px;text-align: center;'>
      <v-btn
        style='width: 100%;margin:20px 0;padding: 0'
        class='primary'
        @click='generatePDF'
      >Скачать печатную форму M-29</v-btn>
    </v-row>
    <div
      id='appPDF'
    >
      <v-row style='width: 100%'>
        <v-col
          cols='6'
          style='text-align: left;'
        >
          <p>«Согласовано»</p>
          <p>Location: {{ m_location }}</p>
          <p>Менеджер проекта</p>
          <p>ТОО "Консорциум ISKER"</p>
          <p>{{ m_manager }} / _________________</p>
          <p>«___»_____________202__г</p>
        </v-col>
        <v-col
          cols='6'
          style='text-align: right;'
        >
          <p>форма М29.2</p>
          <p><strong>«Утверждаю»</strong></p>
          <p>Генеральный Директор</p>
          <p>ТОО "Консорциум ISKER"</p>
          <p>___________ Балгимбаев Б.</p>
          <p>«____»_______________202__г</p>
        </v-col>
      </v-row>
      <v-row style='text-align: center;width: 100%'>
        <h5>О Т Ч Е Т</h5>
        <h5>О РАСХОДЕ МАТЕРИАЛОВ НА ВЫПОЛНЕНИЕ СМР</h5>
        <p style='text-align: center;width: 100%;font-size: 11px;'>Date: {{ m_date }}</p>
        <p style='text-align: center;width: 100%;font-size: 11px;'>Location: {{ m_location }}, {{ m_storage }}, {{ m_placement }}</p>
      </v-row>
      <v-row style='width: 100%'>
        <v-data-table
          :headers="headers"
          :items="goodsWithIndex"
          :items-per-page='-1'
          dense
          hide-default-footer
        >
        </v-data-table>
      </v-row>
      <v-row
        v-for="(item, i) in responsibles" :key="i"
        style='width: 100%'
      >
        <p class='p-list'>{{ item.position }}: {{ item.responsible }} /__________________</p>
      </v-row>
    </div>
    <v-row style='margin: 0;padding: 0'>
      <v-col cols='12'>
        <v-btn
          id='print_button'
          style='width: 800px;margin:20px;'
          class='primary'
          @click="generatePDF()"
        >Скачать печатную форму M-29</v-btn>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: 'M29',
  middleware: 'auth',
  data: () => ({
    contentRendered: false,
    progress: 0,
    generatingPdf: false,
    pdfDownloaded: false,
    headers: [
      { text: '№ п/п', value: 'index', sortable: false, class: "noWrap mini", cellClass: "mini"},
      { text: 'Код', value: 'itemCode', sortable: false, class: "noWrap addWidthMid"},
      { text: 'Наименование материалов', value: 'itemName', sortable: false, class: "noWrap addWidth"},
      { text: 'Ед.изм.', value: 'measure', sortable: false, },
      { text: 'Остаток на начало\n' + 'месяца', value: 'inStock', sortable: false, class: "addWrap"},
      { text: 'Приход', value: 'income', sortable: false, cellClass: "mini2", class: "mini2"},
      { text: 'Расход', value: 'writeoff', sortable: false,cellClass: "mini2", class: "mini2"},
      { text: 'Остаток на конец\n' + 'месяца', value: 'substract', sortable: false, class: "addWrap"},
      { text: 'Примечание', value: 'note', sortable: false, class: "noWrap addWidthMini"},
      { text: 'Баланс / Забаланс', value: 'balance', sortable: false, class: "noWrap"},
    ],
    goods: [],
    responsibles: [],

    dateForm: '', // передаем через роутер выбранную дату списания
    placementForm: '', // передаем через роутер название помещения

    nameForm: 'Списание_MAWS',
    m_date: '',
    m_location: '',
    m_storage: '',
    m_placement: '',
    m_manager: '',
    docName: '',

  }),
  head:{
    title: 'M29 Form',
  },
  computed: {
    user() {
      return this.$auth.user
    },
    goodsWithIndex() {
      return this.goods.map((items, index) => ({...items, index: index + 1 }))
    },
  },
  mounted(){
    this.initialize()
  },
  methods:{
    async initialize() {
      this.dateForm = this.$route.query.date
      this.placementForm = this.$route.query.placement

      try {
        this.out = await this.$axios.$get('api/v1/formsmaws/', {
          params: {
            id_user: this.user.id,
            date: this.dateForm,
            placement: this.placementForm,
            name: this.nameForm,
          }
        })
        this.goods = JSON.parse(this.out[0].goods)
        this.responsibles = JSON.parse(this.out[0].responsibles)

        this.m_date = this.out[0].date
        this.m_manager = this.out[0].manager

        this.m_location = this.out[0].location
        this.m_storage = this.out[0].storage
        this.m_placement = this.out[0].placement

        // Перебираем массив и берем только где есть списание
        this.out = []
        for (let i = 0; i < this.goods.length; i++) {

          // if (this.goods[i].writeoff > 0) {
            if (this.goods[i].balance === 'Нет') {
              this.goods[i].balance = 'Баланс'
            } else {
              this.goods[i].balance = 'Забаланс'
            }

            if (this.goods[i].writeoff === 0) {
              this.goods[i].writeoff = ''
            }

            this.out.push(this.goods[i])
          // }

        }
        this.goods = this.out

        // М29_за_Декабрь 2021 г__MSA_TengizIssue (Проект_склад_помещение)
        let stringDate = this.m_date.substr(6)
        const month = parseInt(this.m_date.substr(4, 2))
        stringDate = this.getMonth(month) + '_' + stringDate + '_г'
        this.docName = 'M29_за_' + stringDate + '__' + this.m_location + '_' + this.m_storage + '_' + this.m_placement + '.pdf'
        this.docName = this.docName.replace(/\s+/g, '')

        console.log('Data initialized')
      } catch (e) {
        console.log(e)
      }
      this.goods = this.goods.map((items) => ({ ...items, substract: items.inStock - items.writeoff }))
      setTimeout(() => { this.generatePDF() }, 1000)
    },

    generatePDF() {
      document.title = this.docName
      window.print()
    },

    getMonth(month){
      const arr=[
        'Январь',
        'Февраль',
        'Март',
        'Апрель',
        'Май',
        'Июнь',
        'Июль',
        'Август',
        'Сентябрь',
        'Октябрь',
        'Ноябрь',
        'Декабрь',
      ];
      return arr[month-1]
    },

    closeTab(){
      close();
    },
  },
}
</script>

<style type='text/css'>
@page
{
  size: A4 landscape;   /* auto is the initial value */
  margin: 10mm;  /* this affects the margin in the printer settings */
}
#appPDF .row {
  width: 800px;
  margin: 0;
}
#appPDF .col {
  width: 400px;margin: 0;padding: 0;
}
#appPDF p {
  font-family: Arial, serif;
  font-size: 12px;
  color: black;
  margin: 0;
  padding: 0;
}
#appPDF h5 {
  font-family: Arial, serif;
  font-weight: bold;
  font-size: 12px;
  width: 100%;
  color: black;
  margin: auto;
  padding: 0;
}
#appPDF .v-data-table {
  width: 100%;
  margin: 10px auto;
  padding: 0;
  text-align: center;
  border-radius: 0;
}

#appPDF .v-data-table .noWrap{
  white-space: nowrap;
}
#appPDF .v-data-table .addWidth{
  width: 380px!important;
}
#appPDF .v-data-table .addWidthMid{
  width: 120px;
}
#appPDF .v-data-table .addWidthMini{
  width: 110px;
}

#appPDF .v-data-table .addWrap{
  width: 80px;
}

#appPDF .v-data-table .mini{
  min-width: 40px;
  width: 50px;
}
#appPDF .v-data-table .mini2{
  min-width: 60px;
  width: 80px;
}

#appPDF .v-data-table table{
  border-collapse: collapse;
  border-spacing: 0;
}

#appPDF table thead tr th, #appPDF table tbody tr td{
  color: black;
  font-size: 12px;
  font-family: Arial, serif;
  font-weight: bold;
  padding: 2px;
  text-align: center;
  border-right: 0.5px solid black;
  border-bottom: 0.5px solid black;
  border-collapse: collapse;
  border-radius: 0;
  min-height: 16px;
  height: 16px;
  line-height: 14px;
  vertical-align: center;
  padding-bottom: 2px!important;
  min-width: 100px;
}

#appPDF table thead tr th {
  border-top: 0.5px solid black;
}
#appPDF table tbody tr td:first-child, #appPDF table thead tr th:first-child {
  border-left: 0.5px solid black;
}
#appPDF .p-list{
  line-height: 16px;
  font-size: 12px;
}

header {
  display: none!important;
  /*@media print {*/
  /*  display: none!important;*/
  /*}*/
}
footer {
  display: none!important;
}
nav {
  display: none!important;
}
main{
  padding: 0!important;
  text-align: center!important;
}
.container {
  padding: 0!important;
  margin: 0!important;
  width: 100%!important;
  max-width: 100%!important;
  text-align: center!important;
}
#appPDF {
  margin: 0 auto;
  padding-top: 50px;
  width: 1200px;
  /*height: 750px;*/
  height: auto;
  /*border: 1px solid lightslategray;*/
}

#print_button{
  display: none;
}


</style>
