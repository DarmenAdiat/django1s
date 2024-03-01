<template>
  <v-container>
    <v-row
      justify="center"
      class="mt-4"
    >
      <v-col
        cols="12"
        sm="10"
        md="10"
        lg="10"
      >
        <v-row>
          <v-col cols='12' lg='8' md='8' sm='8'>
            <h1>Перемещение ОС</h1>
            <h3>Сотрудник: {{ user.first_name }} {{ user.last_name }}</h3>
            <h3>Подразделение-Отправитель: {{ $route.query.selectedCostOtpr }}</h3>
            <h3>Организация-Отправитель: {{ $route.query.selectedOrgOtpr }}</h3>
          </v-col>
          <v-col cols='12' lg='4' md='4' sm='4' style='text-align: right;'>
            <v-btn
              class='primary'
              style='height: 40px !important; font-size: 10px; border-radius: 5px; margin: 55px 5px 5px 5px;'
              @click='goBack'
            >
              <v-icon
                left
                dark
              >
                arrow_back_ios
              </v-icon>
              Назад
            </v-btn>
            <v-btn
              class='primary'
              style='height: 40px !important; font-size: 10px; border-radius: 5px; margin: 55px 5px 5px 5px;'
              @click='sendall'
            >
              Отправка
              <v-icon
                right
                dark
              >
                arrow_forward_ios
              </v-icon>
            </v-btn>
          </v-col>
        </v-row>

        <v-spacer class="mt-5"></v-spacer>
<!--        <v-form-->
<!--          ref="form"-->
<!--          v-model="valid"-->
<!--          @submit.prevent="someAction()"-->
<!--        >-->

<!--        </v-form>-->
        <v-data-table
          :headers='headers'
          :items="this.OSKI"
          :search="search"
          :page.sync="pagination.page"
          item-key="inventory_num"
          v-model='selectedItems'
          show-select
          :single-select='false'
          :itemsPerPage.sync="pagination.itemsPerPage"
          :loading="!this.OSKI.length"
          loading-text="Загрузка..."
          :expanded.sync="expanded"
          show-expand
          single-expand
          @item-expanded="clickExpand"
          style="cursor: pointer"
        >
          <template v-slot:[`item.selection`]="{ item }">
            <v-checkbox v-model="item.selected"></v-checkbox>
          </template>
          <template v-slot:[`item.index`]="{index}">
            {{ (pagination.page - 1) * pagination.itemsPerPage + index + 1 }}
          </template>
          <template v-slot:expanded-item="{headers}">
            <td :colspan="headers.length" align="center">
              <template v-if="os_photo === 'loading'">
                <v-img :src="require('assets/Loading.gif')" width="100" height="100" style="margin: 20px"></v-img>
              </template>
              <template v-if="os_photo != 'loading'">
                <nuxt-img v-bind:src="'data:image/jpeg;base64,' + os_photo" width="500px" style="margin-top: 10px"
                          alt=" no photo"/>
              </template>
            </td>
          </template>
        </v-data-table>
        <v-btn @click='showitems'></v-btn>
      </v-col>
<!--      <v-btn @click='sbordannyh'>Show</v-btn>-->
<!--      <v-btn @click='sendall'>Отправка в 1с</v-btn>-->
    </v-row>
    <v-dialog
      v-model='dialog_success'
      hide-overlay
      persistent
      width='400'
    >
      <v-card
        color='success'
        dark
      >
        <v-card-text style='text-align:center;padding: 24px 20px;color: white;'>
          <h2 style='margin:5px;'>Успешно!</h2>
          <h3 style='margin:5px;'>Ваше заявление на перемещение отправлено</h3>
          <v-btn
            width='100%'
            depressed
            color='info'
            @click='goback'
          >
            Вернуться
          </v-btn>
        </v-card-text>
      </v-card>
    </v-dialog>
    <v-snackbar
      v-model='snack'
      :timeout='3000'
      :color='snackColor'
    >
      {{ snackText }}

      <template #action='{ attrs }'>
        <v-btn
          v-bind='attrs'
          text
          @click='snack = false'
        >
          Закрыть
        </v-btn>
      </template>
    </v-snackbar>
  </v-container>
</template>

<script>
// import moment from 'moment'

const { v4: uuidv4 } = require('uuid');
uuidv4();

export default {
  name: "Send_Os",
  components: {},
  middleware: 'auth',
  data: vm => ({
    loadingField: true,
    OSKI: [],
    POSKI: [],
    os_photo: '',
    headers: [
      {text: '#', value: 'index', sortable: false},
      {text: 'Наименование', value: 'name'},
      {text: 'Инвентарный номер', value: 'inventory_num', sortable: false},
      // {text: 'Серийный номер', value: 'serial_num', sortable: false},
      {text: 'Группа ОС', value: 'os_group'},
      {text: 'МОЛ', value: 'mol'},
      {text: 'Местонахождение', value: 'location'},
      {text: '', value: 'data-table-expand'},
      // {text: 'Выбрать', value: 'selection', sortable: false},
    ],
    search: '',
    pagination:{
      page: 1,
      itemsPerPage: 15
    },
    selectedItems: [],
    expanded: [],
    result1c: '',
    basa64: '',
    snack: false,
    snackColor: '',
    snackText: '',
    dialog_success: false,
  }),
  head: {
    title: 'MoveOs',
  },
  computed: {
    // возвращаем все данные о текущем пользователе
    user() {
      return this.$auth.user
    },
  },

  watch: {

  },
  created() {
    this.getOS()
  },
  beforeMount(){

  },

  methods: {
    async getOS() {
      try {
        this.OSKI = await this.$axios.$get('http://79.143.20.160:3030/HttpService/hs/Web1C/V9', {
          params: {
            iin_num: this.user.id_num,
            // iin_num: '861004300558',
            // CostName: this.$route.query.selectedCostOtpr,
            CostCode: '00-000096'
          },
        })
        if(this.OSKI === 'По этому ИИН нет ОС'){
          this.cancelLoading = true
          this.snack = true
          this.snackColor = 'error'
          this.snackText = 'По вашему ИИН не найдено ОС'
        }
        // console.log(this.OSKI)
        return this.OSKI
      }
      catch (e) {
        this.dialog_success = true
      }
    },
    // функция перезагрузки страницы
    refresh(){
      window.location.reload()
    },
    goback(){
      this.$router.push({
        path: '/MoveOS',
      })
    },
    sbordannyh(){
      const queryParams = {
        type: 0,
        selectedEmpOtpr: this.$route.query.selectedEmpOtpr,
        selectedEmpIINOtpr: this.$route.query.selectedEmpIINOtpr,
        selectedCostOtpr: this.$route.query.selectedCostOtpr,
        selectedCostCodeOtpr: this.$route.query.selectedCostCodeOtpr,
        selectedEmpPol: this.$route.query.selectedEmpPol,
        selectedEmpIINPol: this.$route.query.selectedEmpIINPol,
        adres: 'test',
        Comment: 'test',
        oski: this.selectedItems
        // selectedEmpIINPol: ,
      }
      console.log(queryParams)
    },
    showitems() {
      const table = this.renamefields()
      console.log(table.length)
      return true
    },
    renamefields(){
      console.log(this.selectedItems)
      this.selectedItems = this.selectedItems.map((e) => {
        return {
          Code: e.inventory_num,
          Name: e.name
        };
      });
      console.log(this.selectedItems)
      return this.selectedItems
    },
    async sendall(){
      const table = this.renamefields()
      console.log(table)
      if(table.length>=1){
      this.result1c = await this.$axios.$post('http://nls-1c/1cerp/hs/os/doc/update/', {
        type: 0,
        uid: '',
        NameMol: this.$route.query.selectedEmpOtpr,
        iinMol: this.$route.query.selectedEmpIINOtpr,
        NameCost: this.$route.query.selectedCostOtpr,
        CodeCost: this.$route.query.selectedCostCodeOtpr,
        NameMolTarget: this.$route.query.selectedEmpPol,
        iinMolTarget: this.$route.query.selectedEmpIINPol,
        adres: '',
        Comment: 'test',
        Table: table,
      }, {
        auth:{
          username:'web',
          password:'web',
        }
      })
      // console.log(this.result1c)
      const uid1c = this.result1c.uid
      if(this.result1c){
        this.basa64 = await this.$axios.get('http://nls-1c/1cerp/hs/os/printform/debet/' + uid1c, { auth:{
            username:'web',
            password:'web',
          }})
        console.log(this.basa64)
        const currentDatetime = new Date();
        let formattedDatetime = currentDatetime.toISOString().slice(0, 19).replace('T', ' ');
        const originalDatetime = new Date(formattedDatetime); // Create a copy
        originalDatetime.setHours(originalDatetime.getHours() + 6);
        formattedDatetime = originalDatetime;
        this.result1c = this.$axios.$post('/api/v1/moveos/', {
          type: 0,
          uid: this.result1c.uid,
          EmpOtpr: this.$route.query.selectedEmpOtpr,
          EmpIINOtpr: this.$route.query.selectedEmpIINOtpr,
          CostOtpr: this.$route.query.selectedCostOtpr,
          CostCodeOtpr: this.$route.query.selectedCostCodeOtpr,
          EmpPol: this.$route.query.selectedEmpPol,
          EmpIINPol: this.$route.query.selectedEmpIINPol,
          Address: 'test',
          Comment: 'test',
          Table: table,
          data64: this.basa64.data.data.data64,
          doc_date: formattedDatetime
        })
        this.dialog_success = true
        console.log(table.length)
      }else{
        this.snack = true
        this.snackColor = 'error'
        this.snackText = 'Выберите ОС!'
      }
    }else{
        this.snack = true
        this.snackColor = 'error'
        this.snackText = 'Выберите ОС!'
      }
      },
    goBack(){
      this.$router.push('/CreateOS')
    },
    async clickExpand(item) {
      this.os_photo = 'loading'
      const invnum = item.item.inventory_num
      this.alreadyexpanded = true
      try {
        this.POSKI = await this.$axios.$get('http://79.143.20.160:3030/HttpService/hs/Web1C/V8', {
          params: {
            inv_num: invnum
          },
        })
        this.os_photo = this.POSKI[0].photo
        // console.log('got os')
      } catch (e) {
        this.dialog_error = true
      }
    }
  },
}
</script>

<style scoped>

</style>
