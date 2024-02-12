<template>
  <v-container style='max-width: 100%!important;'>
    <v-btn
        style='bottom:135px;height: 30px;width: 30px;'
        fab
        dark
        fixed
        bottom
        right
        color='black'
        title='Наверх'
        @click="$vuetify.goTo('#up_page', options)"
    >
      <v-icon>keyboard_arrow_up</v-icon>
    </v-btn>
    <v-btn
        style='bottom:100px;height: 30px;width: 30px;'
        fab
        dark
        fixed
        bottom
        right
        color='black'
        title='Вниз'
        @click="$vuetify.goTo('#down_page', options)"
    >
      <v-icon>keyboard_arrow_down</v-icon>
    </v-btn>
    <v-row  id='up_page'>
      <v-col cols='12' lg='6' md='6' sm='12'>
        <h5 style='text-align: left;'>Выбранная дата списания: {{ date_form }}</h5>
        <h5 style='text-align: left;'>Location: {{ $route.query.location }} || {{ $route.query.storage }} || {{ $route.query.placement }}</h5>
        <h5 style='text-align: left;'>Менеджер склада: {{ $route.query.manager }}</h5></v-col>
      <v-col cols='12' lg='6' md='6' sm='12' style='text-align: right;width:100%;padding-top: 20px;padding-right: 30px;'>
        <v-btn
            class='grey darken-1 white--text'
            style='height: 40px !important; font-size: 10px; border-radius: 5px; margin: 5px;width: 20%;'
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
            v-if='isEditing'
            class='green white--text'
            style='height: 40px !important; font-size: 10px; border-radius: 5px; margin: 5px;width: 35%;'
            @click='createTemplate'
        >
          Сохранить
          <v-icon
              right
              dark
          >
            save
          </v-icon>
        </v-btn>
        <v-btn
            v-if='saveTemplate===true'
            class='red white--text'
            style='height: 40px !important; font-size: 10px; border-radius: 5px; margin: 5px;width: 30%;'
            @click='refreshMaws'
        >
          Сбросить
          <v-icon
              right
              dark
          >
            refresh
          </v-icon>
        </v-btn>
      </v-col>
    </v-row>
    <v-row>
      <v-col
          cols='12'
          sm='12'
          md='12'
          lg='12'
          style='margin: auto;'
      >
        <v-data-table
            :headers='headers'
            :items='goods'
            :items-per-page='-1'
            :search='search'
            dense
            class='elevation-1 mawsTable'
            :item-class='itemRowBackground'
            hide-default-footer
            :header-props='headerProps'
            :loading='loadTable'
            loading-text='Загрузка данных'
        >
          <template #[`item.index`]='{ item }'>
            {{ goods.indexOf(item)+1 }}
          </template>
          <template #top>
            <v-toolbar
                flat
                style='margin-bottom: 20px;'
            >
              <v-text-field
                  v-model='search'
                  append-icon='mdi-magnify'
                  label='Поиск'
                  single-line
                  hide-details
              ></v-text-field>
              <v-divider
                  class='mx-4'
                  inset
                  vertical
              ></v-divider>
              <v-spacer></v-spacer>
              <v-toolbar-items>
                <v-btn
                    v-if='((blockEdit===false) && (isEditing))'
                    id='btnDisposal'
                    class='primary'
                    style='height: 40px !important; font-size: 10px; border-radius: 5px; margin: 5px;'
                    @click='show_cost_center'
                >Показать/Скрыть Подразделение-получатель
                </v-btn>
                <v-btn
                    v-if='blockEdit===false'
                    id='btnDisposal'
                    class='primary'
                    style='height: 40px !important; font-size: 10px; border-radius: 5px; margin: 5px;'
                    @click='startDisposal'
                >{{ isEditing ? 'Отменить списание' : 'Приступить к списанию' }}
                  <v-icon
                      v-if='isEditing'
                      right
                      dark
                  >
                    close
                  </v-icon>
                  <v-icon
                      v-if='!isEditing'
                      right
                      dark
                  >
                    mdi-plus
                  </v-icon>
                </v-btn>
                <v-btn
                    v-if='blockEdit===true'
                    id='btnDisposal'
                    class='red'
                    disabled
                    style='height: 40px !important; font-size: 10px; border-radius: 5px; margin: auto;color:white'
                >В данный момент списывает {{ statusLine }}
                </v-btn>
              </v-toolbar-items>
            </v-toolbar>
          </template>

          <template
              v-if="$route.query.cost==='true'"
              #[`item.cos_center`]='{ item }'>
            <v-select
                v-if='isEditing===true'
                v-model='item.cos_center'
                class="select_cos"
                :items='cons'
                item-text='name'
                return-object
                style="width:160px!important;font-size: 10px!important;"
                :error-messages='selectErrors[goods.indexOf(item)]'
                dense
                outlined
                @change='editItem(item)'
                @keyup.enter='$event.target.blur()'
            ></v-select>
          </template>

          <template #[`item.writeoff`]='{ item }'>
            <v-text-field
                v-show='isEditing===true'
                id='inputDisposal'
                v-model='item.writeoff'
                class='inputDisposal'
                type='number'
                outlined
                @blur='editItem(item)'
                @keyup.enter='$event.target.blur()'
            ></v-text-field>
          </template>
          <template #no-data>
            <v-btn
                color='primary'
                style='margin-bottom: 10px;'
                @click='initialize'
            >
              Обновить
            </v-btn>
          </template>
          <template #no-results>
            <v-alert :value='true' color='error' icon='warning'>
              По вашему поиску "{{ search }}" не найдено результатов.
            </v-alert>
          </template>
        </v-data-table>
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
      </v-col>
    </v-row>
    <v-row v-show='isEditing===true' style='margin-top: 50px;text-align: center'>
      <v-col lg='6' md='6' sm='12'>
        <v-btn
            width='100%'
            height='70'
            class='grey darken-1 white--text'
            @click='downloadM29'
        >Сформировать документ M-29
        </v-btn>
      </v-col>
      <v-col lg='6' md='6' sm='12'>
        <v-btn
            width='100%'
            height='70'
            class='grey darken-1 white--text'
            @click='downloadZ6'
        >Сформировать документ Z-6
        </v-btn>
      </v-col>
    </v-row>
    <v-row v-show='isEditing===true'>
      <v-col cols='12' lg='4' md='4' sm='12' style='margin: auto;'>
        <h4 class='maws_fileinputs_h4'>Прикрепить скан формы M-29</h4>
        <v-file-input
            v-model='files'
            :rules='fileRules'
            accept='.pdf'
            color='red darken-4'
            label='Прикрепить форму M-29'
            placeholder='Прикрепите ваши файлы'
            prepend-icon='mdi-paperclip'
            outlined
            :show-size='1000'
            required
        >
          <template #selection='{ index, text }'>
            <v-chip
                v-if='index < 3'
                color='black'
                dark
                label
                small
            >
              {{ text }}
            </v-chip>
          </template>
        </v-file-input>
      </v-col>
      <v-col cols='12' lg='4' md='4' sm='12' style='margin: auto;'>
        <h4 class='maws_fileinputs_h4'>Прикрепить скан формы Z-6</h4>
        <v-file-input
            v-model='files_2'
            :rules='fileRules'
            accept='.pdf'
            color='red darken-4'
            label='Прикрепить форму Z-6'
            placeholder='Прикрепите ваши файлы'
            prepend-icon='mdi-paperclip'
            outlined
            :show-size='1000'
            required
        >
          <template #selection='{ index, text }'>
            <v-chip
                v-if='index < 3'
                color='black'
                dark
                label
                small
            >
              {{ text }}
            </v-chip>
          </template>
        </v-file-input>
      </v-col>
      <v-col cols='12' lg='4' md='4' sm='12' style='margin: auto;'>
        <h4 class='maws_fileinputs_h4'>Дополнительные файлы, при наличии</h4>
        <v-file-input
            v-model='files_3'
            accept='image/*,.pdf,.xlsx,.xls'
            color='red darken-4'
            counter
            label='Дополнительне документы'
            multiple
            placeholder='Прикрепите ваши файлы'
            prepend-icon='mdi-paperclip'
            outlined
            :show-size='1000'
        >
          <template #selection='{ index, text }'>
            <v-chip
                v-if='index < 4'
                color='black'
                dark
                label
                small
            >
              {{ text }}
            </v-chip>
          </template>
        </v-file-input>
      </v-col>
    </v-row>
    <v-row v-show='isEditing===true'>
      <v-col lg='3' md='3' sm='12' style='margin: auto;'>
        <v-btn
            width='100%'
            height='70'
            depressed
            class='grey darken-1 white--text'
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
      </v-col>
      <v-col lg='3' md='3' sm='12' style='margin: auto;'>
        <v-btn
            width='100%'
            height='70'
            depressed
            class='green white--text'
            @click='createTemplate'
        >
          Сохранить черновик
          <v-icon
              right
              dark
          >
            save
          </v-icon>
        </v-btn>
      </v-col>
      <v-col lg='6' md='6' sm='12' style='margin: auto;'>
        <v-btn
            id='down_page'
            width='100%'
            height='70'
            depressed
            class='primary'
            @click='checkForm'
        >
          Отправить данные на списание
          <v-icon
              right
              dark
          >
            send
          </v-icon>
        </v-btn>
      </v-col>
    </v-row>
    <v-spacer style='height: 200px;'></v-spacer>
    <v-dialog
        v-model='dialog_validate'
        hide-overlay
        persistent
        width='400'
    >
      <v-card
          color='yellow'
          dark
      >
        <v-card-text style='text-align:center;padding: 24px 20px;color: black;'>
          <h2 style='margin:5px;'>Ошибка при заполнениее формы!</h2>
          <h3 style='margin:5px;'>Форма не отправлена</h3>
          <h3 style='margin:20px;'>{{ error }}</h3>
          <v-btn
              width='100%'
              depressed
              color='black'
              @click='dialog_validate=false'
          >
            Продолжить заполнение формы
          </v-btn>
        </v-card-text>
      </v-card>
    </v-dialog>
    <v-dialog
        v-model='dialog_success'
        hide-overlay
        persistent
        width='400'
    >
      <v-card
          color='green'
          dark
      >
        <v-card-text style='text-align:center;padding: 24px 10px;'>
          <h2 style='margin:20px;color: white;'>Ваше заявление на списание<br>отправляется...</h2>
          <v-progress-linear
              indeterminate
              color='white'
              style='margin: 20px 5px 20px 5px; height:10px; width: auto;'
          ></v-progress-linear>
        </v-card-text>
      </v-card>
    </v-dialog>
    <v-dialog
        v-model='dialog_error'
        hide-overlay
        persistent
        width='400'
    >
      <v-card
          color='red'
          dark
      >
        <v-card-text style='text-align:center;padding: 24px 20px;color: white;'>
          <h2 style='margin:5px;'>Ошибка на сервере 1С</h2>
          <h3 style='margin:5px;'>Ваше заявление на списание не отправлено</h3>
          <h3 style='margin:20px;'>Попробуйте еще раз<br>или обратитесь в службу поддержки</h3>
          <v-btn
              width='100%'
              depressed
              color='black'
              @click='refresh'
          >
            Перезагрузить страницу
          </v-btn>
        </v-card-text>
      </v-card>
    </v-dialog>
  </v-container>
</template>

<script>

import moment from 'moment'

const { v4: uuidv4 } = require('uuid')
uuidv4()

export default {
  name: 'Maws',
  // ПРЕДУПРЕЖДЕНИЕ О ПЕРЕХОДЕ НА ДРУГУЮ СТРАНИЦУ
  beforeRouteLeave(to, from, next) {
    if ((this.saveTemplate === false) && (this.isEditing === true)) {
      if (!window.confirm('ПРЕДУПРЕЖДЕНИЕ!\nПодтвердите переход на другую страницу!\nНесохраненные данные будут утеряны')) {
        return
      }
    }
    next()
  },
  middleware: 'auth',
  data() {
    return {
      cost_show: false,
      selectErrors: [], // ошибка валидации на селекторе кос центров
      // this.selectErrors = this.$t('snack_currency')
      saveTemplate: false, // отметка о сохранении черновика
      nameForm: 'Списание_MAWS',
      idUUID: uuidv4(),
      dateToday: moment(new Date()).format('yyyy-MM-DD HH:mm:ss'), // Текущая дата

      m_id: '',
      m_date: '',
      m_manager: '',
      m_location: '',
      m_storage: '',
      m_placement: '',

      snack: false,
      snackColor: '',
      snackText: '',
      valid: true,
      dialog: false,
      dialog_success: false,
      dialog_error: false,
      dialog_validate: false,
      mobile: true,
      loadTable: true, // Загрузка таблицы лоадер
      error: '',
      statusLine: '-',
      isEditing: false,  // СОСТОЯНИЕ РЕДАКТИРОВАНИЯ // вкл(тру) или выкл(фолс) // Добрать через API пользователя из БД
      blockEdit: false, // проверка на блокировку редактирование другим юзером
      out: [],
      files: [], // files 1st input
      files_2: [], // files 2nd input
      files_3: [], // files 3rd input
      fileRules: [
        value => !!value || 'Необходимо прикрепить файл',
        value => (value && value.size > 0) || 'Необходимо прикрепить файл',
        value => !value || value.size < 8000000 || 'Максимальный размер файла 8МБ',
      ],
      doc_status: 6, // первоначальный статус документа - черновик
      doc_name: 'Заявка на списание', // название документа в бд
      duration: 1000,
      offset: 0,
      easing: 'easeInOutCubic',
      search: '', // поисковик
      writeoffRules: [
        v => !!v || 'Введите значение',
        v => (v >= 0) || 'Значение не может быть меньше 0',
      ],
      headerProps: {
        sortByText: 'Сортировать таблицу'
      },
      // headers: [
      //   { text: '#', value: 'index', sortable: false },
      //   { text: this.$t('goods_code'), value: 'itemCode', sortable: false },
      //   { text: this.$t('goods_name'), value: 'itemName', cellClass: 'cellCustom2' },
      //   { text: this.$t('goods_name_en'), value: 'itemName_en' },
      //   { text: this.$t('goods_purpose'), value: 'purpose', sortable: false },
      //   { text: this.$t('goods_measure'), value: 'measure', sortable: false },
      //   { text: this.$t('goods_stock'), value: 'inStock', sortable: false, cellClass: 'cellCustom' },
      //   { text: this.$t('goods_blocked'), value: 'nowBlocked', cellClass: 'cellCustom', class: 'cellCustom', sortable: false },
      //   { text: this.$t('goods_writeoff'), value: 'writeoff', class: 'cellCustomHead', cellClass: 'cellCustom' }
      // ],
      goods: [],
      date_form: '',
      placement_form: '',
      responsibles: [],
      cons: [],
      state: [],
      editedIndex: -1,
      editedItem: {
        measure: '',
        inStock: 0,
        nowBlocked: 0,
        writeoff: 0
      }
    }},
  head: {
    title: 'MAWS'
  },
  computed: {
    options() {
      return {
        duration: this.duration,
        offset: this.offset,
        easing: this.easing
      }
    },
    user() {
      return this.$auth.user
    },
    headers(){
      if (this.$route.query.cost === 'true') {
        const tableHead = [
          { text: '#', value: 'index', sortable: false },
          { text: 'Код товара', value: 'itemCode', sortable: false },
          { text: 'Номенклатура', value: 'itemName', cellClass: 'cellCustom2' },
          { text: 'Item', value: 'itemName_en' },
          { text: 'Назначение', value: 'purpose', sortable: false },
          { text: 'Ед.изм.', value: 'measure', sortable: false },
          { text: 'Подразделение', value: 'cos_center', sortable: false },
          { text: 'Остаток', value: 'inStock', sortable: false, cellClass: 'cellCustom' },
          { text: 'Ожидает списания', value: 'nowBlocked', cellClass: 'cellCustom', class: 'cellCustom', sortable: false },
          { text: 'Списание', value: 'writeoff', class: 'cellCustomHead', cellClass: 'cellCustom' }
        ]
        return tableHead
      } else {
        const tableHead = [
          { text: '#', value: 'index', sortable: false },
          { text: 'Код товара', value: 'itemCode', sortable: false },
          { text: 'Номенклатура', value: 'itemName', cellClass: 'cellCustom2' },
          { text: 'Item', value: 'itemName_en' },
          { text: 'Назначение', value: 'purpose', sortable: false },
          { text: 'Ед.изм.', value: 'measure', sortable: false },
          { text: 'Остаток', value: 'inStock', sortable: false, cellClass: 'cellCustom' },
          { text: 'Ожидает списания', value: 'nowBlocked', cellClass: 'cellCustom', class: 'cellCustom', sortable: false },
          { text: 'Списание', value: 'writeoff', class: 'cellCustomHead', cellClass: 'cellCustom' }
        ]
        return tableHead
      }
    }
  },
  watch: {
    dialog(val) {
      val || this.close()
    }
  },
  // ПРЕДУПРЕЖДЕНИЕ О ПЕРЕЗАГРУЗКЕ СТРАНИЦЫ
  beforeMount() {
    window.addEventListener('beforeunload', this.preventNav)
    this.$once('hook:beforeDestroy', () => {
      window.removeEventListener('beforeunload', this.preventNav)
    })
  },
  created() {
    this.initialize()
    this.getResponsibles() // ВКЛЮЧИТЬ И ПОДОГНАТЬ ССЫЛКУ
    this.getCons() // получить список актуальных кост центров
    this.getMawsInfo()
  },
  methods: {
    async initialize() {

      // ==================================================
      // start of activities
      this.date_form = moment(this.$route.query.date).format('DD.MM.yyyy')
      this.placement_form = this.$route.query.placement

      // TRY TO GET DATA FROM TEMPLATE
      try {
        this.out = await this.$axios.$get('api/v1/formsmaws/', {
          params: {
            id_user: this.user.id,
            date: this.date_form,
            placement: this.placement_form,
            name: this.nameForm
          }
        })
        this.goods = JSON.parse(this.out[0].goods)
        this.responsibles = JSON.parse(this.out[0].responsibles)


        // console.log("sort 1")
        // console.log(this.out2)
        // console.log("sort 2")

        this.m_id = this.out[0].id
        this.m_date = this.out[0].date
        this.m_manager = this.out[0].manager

        this.m_location = this.out[0].location
        this.m_storage = this.out[0].storage
        this.m_placement = this.out[0].placement

        // console.log('url_id: ' + this.m_id)
        // console.log('url_date: ' + this.date_form)
        // console.log('url_placement: ' + this.placement_form)

        // this.docName = 'M29_' + this.m_date + '_' + this.m_location + '_' + this.m_storage + '_' + this.m_placement + '.pdf'
        // this.docName = this.docName.replace(/\s+/g, '')
        // console.log(this.docName)
        this.saveTemplate = true
        this.loadTable = false
        console.log('Data initialized')
      } catch (e) {
        this.saveTemplate = false
      }

      if (this.saveTemplate === false) {
        // ПОЛУЧАЕМ ВСЕ ТОВАРЫ ПО СКЛАДУ FROM `1C`
        try {
          this.goods = await this.$axios.$get('http://' + process.env.globalURL + '/HttpService/hs/Web1C/V1', {
            // this.goods = await this.$axios.$get('http://' + process.env.globalURL + '/testweb/hs/Web1C/V1', {
            params: {
              iin_num: this.user.id_num, // отправляем в гет запросе Код выбранного помещения
              placement: this.placement_form, // отправляем в гет запросе Название выбранного помещения
              date: this.$route.query.date // отправляем в гет запросе дату
            }
          })
          console.log('Data initialized')
          this.loadTable = false
          return this.goods // возвращаем полученное значение функции
        } catch (e) {
          console.log(e)
        }
      }
      // end of activities
      // ==================================================
    },

    async getResponsibles() {
      this.responsibles = []
      // ПОЛУЧАЕМ ОТВЕТСТВЕННЫХ ЗА ПОМЕЩЕНИЕ/СКЛАД
      try {
        this.responsibles = await this.$axios.$get('http://nls-1c/HttpService/hs/Web1C/V3', {
          // this.responsibles = await this.$axios.$get('http://' + process.env.globalURL + '/testweb/hs/Web1C/V3', {
          params: {
            placement: this.$route.query.placement // отправляем в гет запросе Код выбранного помещения
          }
        })
        // console.log(this.responsibles)
        // console.log('Got Responsibles')
        // console.log(this.responsibles)
        const wordsArray = ['супервайзер', 'суперинтендант']

        const responsiblesOut = []
        responsiblesOut.push(this.responsibles[0])
        responsiblesOut.push(this.responsibles[1])

        for (let i = 2; i < this.responsibles.length; i++) {
          const measurePos = this.responsibles[i].position.toLowerCase()
          const result = wordsArray.some(w => measurePos.includes(w))
          if (result === true) {
            responsiblesOut.push(this.responsibles[i])
          }
        }
        this.responsibles = responsiblesOut
        // console.log(this.responsibles)
        // console.log('Responsibles reformatted')
        console.log('Got Responsibles')
        return this.responsibles // возвращаем полученное значение функции
      } catch (e) {
        console.log(e)
      }
    },
    async getCons() {
      this.cons = []
      // ПОЛУЧАЕМ кост центры
      try {
        // this.responsibles = await this.$axios.$get('http://' + process.env.globalURL + '/HttpService/hs/Web1C/V3', {

        this.cons = await this.$axios.$post('http://nls-1c/1cerp/hs/distribut/getavailablecon/', {
              name: this.$route.query.location
            },
            {
              credentials: true,
              auth: {
                username: 'web',
                password: 'web'
              },
            })
        // this.cons = [
        //     {
        //       name: "TCO Camp Project General",
        //       cod: "-00-00001"
        //     },
        //     {
        //       name: "TCO Camp Project General 2",
        //       cod: "-00-00002"
        //     }
        //    ]
        // console.log(this.cons)
        console.log('Got Cons')
        return this.cons // возвращаем полученное значение функции
      } catch (e) {
        console.log(e)
      }
    },

    async getMawsInfo() {
      try {
        // this.$route.query.storage = "OV.TENGIZ.ISSUE" // Поле СКЛАДА

        // // ПОЛУЧАЕМ СОСТОЯНИЕ ПОМЕЩЕНИЯ - state
        this.state = await this.$axios.$get('api/v1/mawseditstatus/', {
          params: {
            placement: this.$route.query.placement, // отправляем наименование помещения
            storage: this.$route.query.storage // отправляем наименование склада
          }
        })
        console.log('Got MAWS info')
        if (this.state[0]) {
          if (this.state[0].id_user === this.user.id) {
            this.blockEdit = false
            this.statusLine = 'free'
            if (this.isEditing === false) {
              this.isEditing = true
              console.log('Включено редактирование после обновления страницы!!!')
            }
            console.log(this.blockEdit)
            console.log('USER СОВПАЛ')
          } else {
            this.blockEdit = true
            this.statusLine = this.state[0].user
            console.log(this.blockEdit)
            console.log('USER НЕ СОВПАЛ')
          }
        } else {
          this.blockEdit = false
          console.log(this.blockEdit)
          console.log('Нет записей')
        }
      } catch (e) {
        console.log('ОШИБКА')
      }


    },
    // ПЕРЕЗАГРУЗКА СТРАНИЦЫ - ALERT
    preventNav(event) {
      if (!this.isEditing) return
      event.preventDefault()
      event.returnValue = ''
    },
    // КРАСИМ КЛАССОМ ИЗМЕНЕННЫЕ СТРОКИ
    itemRowBackground(item) {
      if ((typeof item.inStock) === 'string') {
        item.inStock = item.inStock.replace(/\s+/g, '')
        item.inStock = item.inStock.replace(',', '.')
        item.inStock = parseFloat(item.inStock)
      }
      if ((typeof item.nowBlocked) === 'string') {
        item.nowBlocked = item.nowBlocked.replace(/\s+/g, '')
        item.nowBlocked = item.nowBlocked.replace(',', '.')
        item.nowBlocked = parseFloat(item.nowBlocked)
      }

      if ((item.writeoff > 0) && (item.writeoff <= (item.inStock - item.nowBlocked))) {
        return 'editedRow'
      } else if (item.writeoff === 0) {
        return ''
      } else {
        return 'editedRowError'
      }
    },
    editItem(item) {
      this.editedIndex = this.goods.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.save()
    },
    save() {
      if (!this.editedItem.nowBlocked) {
        this.editedItem.nowBlocked = 0
      }
      if (this.editedItem.writeoff > (this.editedItem.inStock - this.editedItem.nowBlocked) || this.editedItem.writeoff < 0 || !this.editedItem.writeoff) {
        this.editedItem.writeoff = 0
        Object.assign(this.goods[this.editedIndex], this.editedItem)

        this.snack = true
        this.snackColor = 'error'
        this.snackText = 'Ошибка ввода, не сохранено!'

        // console.log(this.goods)
        return
      }

      // if (!this.editedItem.cos_center) {
      //   this.selectErrors[this.editedIndex] = 'Укажите подразделение'
      //   // add snack error
      // } else {
      //   this.selectErrors[this.editedIndex] = ''
      // }

      if (this.editedIndex > -1) {
        const wordsArray = ['пара', 'упак', 'пачка', 'компл', 'коробка', 'шт']
        const measureString = this.editedItem.measure.toLowerCase()
        const result = wordsArray.some(w => measureString.includes(w))
        // console.log('result:', result, measureString)

        if (result === true) {
          this.editedItem.writeoff = parseInt(this.editedItem.writeoff)
        } else {
          this.editedItem.writeoff = parseFloat(this.editedItem.writeoff)
          this.editedItem.writeoff = Math.round(this.editedItem.writeoff * 1000) / 1000
          // console.log(this.editedItem.writeoff)
        }
        // this.editedItem.writeoff = parseFloat(this.editedItem.writeoff)
        Object.assign(this.goods[this.editedIndex], this.editedItem)

        // if (this.editedItem.cos_center) {
        //   this.selectErrors[this.editedIndex] = ''
        //   this.snack = true
        //   this.snackColor = 'success'
        //   this.snackText = this.$t('data_accepted')
        // } else {
        //   this.selectErrors[this.editedIndex] = this.$t('set_cos_center')
        //   this.snack = true
        //   this.snackColor = 'error'
        //   this.snackText = this.$t('set_cos_center')
        // }


        // console.log(this.goods)
      } else {
        this.goods.push(this.editedItem)

        // if (this.editedItem.cos_center) {
        //   this.selectErrors[this.editedIndex] = ''
        //   this.snack = true
        //   this.snackColor = 'success'
        //   this.snackText = this.$t('data_accepted')
        // } else {
        //   this.selectErrors[this.editedIndex] = this.$t('set_cos_center')
        //   this.snack = true
        //   this.snackColor = 'error'
        //   this.snackText = this.$t('set_cos_center')
        // }
        // console.log(this.goods)
      }
    },
    // ======= Создание и удаление статуса - сейчас редактируется
    async createState() {
      await this.$axios.$post('api/v1/mawseditstatus/', {
        id_user: this.user.id,
        user: this.user.first_name + ' ' + this.user.last_name,
        storage: this.$route.query.storage,
        placement: this.$route.query.placement
      })
      console.log('Запись создана')
    },
    async deleteState() {
      await this.$axios.$delete('api/v1/mawseditstatus/' + this.state[0].id)
      console.log('Запись удалена')

      // добавить сюда удаление черновика

      // добавить сюда удаление черновика
    },
    // ======= Создание и удаление статуса - сейчас редактируется

    async refreshMaws() {
      // Добавить удаление черновика здесь с условием что он есть!
      await this.deleteFormsMaws()
      await this.deleteLogs()
      await this.initialize()
      this.saveTemplate = false
      this.snack = true
      this.snackColor = 'error'
      this.snackText = 'Данные сброшены и обновлены'
    },

    // Открыть доступ к списанию
    async startDisposal() {
      // alert('LETs GET IT STARTED, NIGGA')

      await this.getMawsInfo()

      if (this.isEditing === false) {

        if (this.state[0]) {
          if (this.state[0].id_user === this.user.id) {
            this.blockEdit = false
            this.statusLine = 'free'
            console.log(this.blockEdit)
            console.log('USER СОВПАЛ')
          } else {
            this.blockEdit = true
            this.statusLine = this.state[0].user
            console.log(this.blockEdit)
            console.log('USER НЕ СОВПАЛ')
          }
        } else {
          this.blockEdit = false
          console.log(this.blockEdit)
          console.log('Нет записей')

          // СОЗДАНИЕ ПО НАЖАТИЮ КНОПКИ СПИСАНИЯ и удаление при отжатии - Начало
          await this.createState()

          console.log('Повторный вызов начат')
          await this.getMawsInfo()
          // - завершение
        }
        if (this.blockEdit === false) {
          // alert("DISPOSAL ENABLED")
          this.isEditing = true
        }
      } else {

        // Добавить удаление черновика здесь с условием что он есть!
        if (this.saveTemplate === true) {
          await this.deleteFormsMaws()
          // Добавить удаление черновика здесь
          await this.deleteLogs()
        }
        // удаление при отжатии КНОПКИ СПИСАНИЯ - Начало
        await this.deleteState()
        if (this.blockEdit === false) {
          this.isEditing = false
          this.$router.push(this.localePath('/Mawslist'))
        }
      }
      console.log('Редактируется: ' + this.isEditing)
    },

    // Проверка файлов на вес и на наличие
    checkForm() {
      this.out = []
      if ((!this.files) || (!this.files_2)) {
        this.error = 'Вы не прикрепили скан документа!'
        this.dialog_validate = true
        this.out = []
        return
      } else if (((this.files) && (this.files.length === 0)) || (this.files_2) && (this.files_2.length === 0)) {
        this.error = 'Вы не прикрепили скан документа!'
        this.dialog_validate = true
        this.out = []
        return
      } else if (this.files.size > 8 * 1024 * 1024) {
        this.error = 'Файл Z-6 превышает размер 8МБ!'
        this.dialog_validate = true
        this.out = []
        return
      } else if (this.files_2.size > 8 * 1024 * 1024) {
        this.error = 'Файл Z-6 превышает размер 8МБ!'
        this.dialog_validate = true
        this.out = []
        return
      } else {
        this.error = 'ОК'
        // this.dialog_validate = true

        // Перебираем массив и берем только где есть списание
        for (let i = 0; i < this.goods.length; i++) {
          if (this.goods[i].writeoff > 0)
            this.out.push(this.goods[i])
        }
      }
      return this.submitFile()
    },
    getBase64(file) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result);
        reader.onerror = error => reject(error);
      });
    },

    async submitFile() {

      await this.createTemplate() // обновляем или создаем черновик
      // const headers = { 'Content-Type': 'multipart/form-data' }
      // const formData = new FormData()

      // Перебираем массив и берем только где есть списание - уже перебрали
      this.out = []
      for (let i = 0; i < this.goods.length; i++) {
        if (this.goods[i].writeoff > 0)
          this.out.push(this.goods[i])
      }
      // here sort
      this.out = this.out.sort(function(a, b){
        if (a.purpose < b.purpose) return -1;
        if (a.purpose > b.purpose) return 1;
        return 0;
      })

      // console.log(this.out)

      this.out = JSON.stringify(this.out)

      // id: this.m_id // base64 идентификатор заявки (уникальный ключ)
      // const iinNum1 = this.user.id_num
      // formData.append('id', this.m_id)
      // formData.append('iin_num', iinNum1)
      // formData.append('placement', this.$route.query.placement)
      // formData.append('date', this.$route.query.date)
      // formData.append('goods', this.out)
      // formData.append('fileM29', this.files)
      // formData.append('fileZ6', this.files_2)
      // formData.append('filesAdd', this.files_3)

      // let tempFileM29 = {}
      // const filenameM29 = this.files.name
      // const filebase64M29 = await this.getBase64(this.files)
      // tempFileM29 = { name: filenameM29, file: filebase64M29 }

      // let tempFileZ6 = {}
      // const filenameZ6 = this.files_2.name
      // const filebase64Z6 = await this.getBase64(this.files_2)
      // tempFileZ6 = { name: filenameZ6, file: filebase64Z6 }

      const filesArray = []
      for (let i = 0; i < this.files_3.length; i++) {
        let tempFiles = {}
        const filename = this.files_3[i].name
        const filebase64 = await this.getBase64(this.files_3[i])
        tempFiles = { name: filename, file: filebase64 }
        filesArray.push(tempFiles)
      }


      // try {
      //   this.dialog_success = true
      //   // await this.$axios.$post('http://' + process.env.globalURL + '/HttpService/hs/Web1C/V4', formData, { headers })
      //   await this.$axios.$post('http://nls-1c/HttpService/hs/Web1C/V4', {
      //     id: this.m_id,
      //     iin_num: iinNum1,
      //     placement: this.$route.query.placement,
      //     date: this.$route.query.date,
      //     goods: this.out,
      //     fileM29: tempFileM29,
      //     fileZ6: tempFileZ6,
      //     filesAdd: filesArray
      //   })
      //
      //
      //   // Завернем в ЕСЛИ - если черновик создан то только меняем статус, если не создан то создаем запись
      //   this.doc_status = 2 // Ставим статус - На согласовании
      //   if (this.saveTemplate === true) {
      //     // изменение статуса `черновик` => `На согласовании`
      //     await this.$axios.$patch('api/v1/loglist/' + this.m_id + '/', {
      //       doc_date: this.dateToday,
      //       doc_status: this.doc_status
      //     })
      //     console.log('Статус: На согласовании')
      //   } else {
      //     // Добавить создание черновика здесь
      //
      //     // отправка заявки в БД таблицу логирования
      //     await this.$axios.$post('api/v1/loglist/', {
      //       id: this.m_id, // base64 идентификатор заявки (уникальный ключ)
      //       id_user: this.user.id,
      //       id_num: this.user.id_num,
      //       first_name: this.user.first_name,
      //       last_name: this.user.last_name,
      //       full_name: this.user.full_name,
      //       doc_date: this.dateToday,
      //       doc_name: this.doc_name,
      //       doc_status: this.doc_status,
      //       tag: 'maws'
      //     })
      //   }
      //
      //   // Добавить в черновики запись в название что списание проведено
      //   await this.$axios.$patch('api/v1/formsmaws/' + this.m_id + '/', {
      //     name: this.nameForm + '_DONE'
      //   })
      //
      //   this.snack = true
      //   this.snackColor = 'success'
      //   this.snackText = 'Заявка отправлена'
      //   await new Promise(resolve => setTimeout(() => {
      //     (this.dialog_success = false);
      //     (this.deleteState)(this.isEditing = false);
      //     // (this.deleteFormsMaws()); // удаляем черновик
      //     (this.$router.push(this.localePath('/Mawslist')))
      //   }, 2500))
      // } catch (e) {
      //   this.dialog_success = false
      //   this.dialog_error = true // открывает диалоговое окно об ошибке с кнопкой перезагрузки
      // }
    },

    async downloadM29() {
      // Создаем чернови если не был создан ранее или обновляем
      await this.createTemplate()
      // Открываем в новой вкладке страницу генерации печатной формы М29
      const route = this.$router.resolve({
        path: 'pdf',
        query: {
          date: this.date_form,
          placement: this.placement_form
        }
      })

      window.open(route.href, '_blank')
    },

    // refactor like M29
    async downloadZ6() {
      // Создаем чернови если не был создан ранее или обновляем
      await this.createTemplate()
      // Открываем в новой вкладке страницу генерации печатной формы М29
      const route = this.$router.resolve({
        path: 'pdf_z6',
        query: {
          date: this.date_form,
          placement: this.placement_form
        }
      })
      window.open(route.href, '_blank')
    },

    // Удаление черновика по идентификатору
    async deleteFormsMaws() {
      await this.$axios.$get('api/v1/formsmaws/delete_post', {
        params: {
          id: this.m_id
        }
      })
      console.log('FormMaws Deleted')
    },

    // async deleteLogs() {
    //   await this.$axios.$delete('api/v1/loglist/' + this.m_id + '/')
    //   console.log('Logs Deleted')
    // },

    refresh() {
      window.location.reload()
    },

    // Функция создания черновика
    // async createTemplate() {
    //
    //   this.date_form = moment(this.$route.query.date).format('DD.MM.yyyy')
    //   try {
    //     // Если мы хотим обновить черновик !!!
    //     if (this.saveTemplate === true) {
    //       // если уже было создано, то больше не создаем в логах запись
    //       await this.$axios.$patch('api/v1/formsmaws/' + this.m_id + '/', {
    //         goods: JSON.stringify(this.goods),
    //         responsibles: JSON.stringify(this.responsibles)
    //       })
    //       await this.$axios.$patch('api/v1/loglist/' + this.m_id + '/', {
    //         doc_date: this.dateToday
    //       })
    //     }
    //     // Если черновика ЕЩЕ НЕТ !!!
    //     else {
    //       this.m_id = this.idUUID
    //       // если уже было создано, то больше не создаем в логах запись
    //       await this.$axios.$post('api/v1/formsmaws/', {
    //         id: this.m_id,
    //         id_user: this.user.id,
    //         name: this.nameForm,
    //         goods: JSON.stringify(this.goods),
    //         // goods: JSON.stringify(this.out),
    //         responsibles: JSON.stringify(this.responsibles),
    //         storage: this.$route.query.storage,
    //         placement: this.placement_form,
    //         manager: this.$route.query.manager,
    //         location: this.$route.query.location,
    //         date: this.date_form
    //       })
    //       // отправка заявки в БД таблицу логирования
    //       await this.$axios.$post('api/v1/loglist/', {
    //         id: this.m_id, // base64 идентификатор заявки (уникальный ключ)
    //         id_user: this.user.id,
    //         id_num: this.user.id_num,
    //         first_name: this.user.first_name,
    //         last_name: this.user.last_name,
    //         full_name: this.user.full_name,
    //         doc_name: this.doc_name,
    //         doc_date: this.dateToday,
    //         doc_status: this.doc_status,
    //         tag: 'maws'
    //       })
    //     }
    //     this.saveTemplate = true // отметка о сохранении черновика
    //
    //     this.snack = true
    //     this.snackColor = 'success'
    //     this.snackText = 'Данные сохранены'
    //
    //     console.log('Template Created')
    //   } catch (e) {
    //     console.log('Template ERROR: Not created')
    //   }
    // },
    // end of func

    goBack(){
      this.$router.push(this.localePath('/Mawslist'))
    },


    show_cost_center(){
      if (this.$route.query.cost !== 'true') {
        this.cost_show = true
        this.$router.push({
          path: 'Maws',
          query: {
            placement: this.$route.query.placement,
            storage: this.$route.query.storage,
            location: this.$route.query.location,
            manager: this.$route.query.manager,
            date: this.$route.query.date,
            cost: this.cost_show,
          }
        });
      } else {
        this.cost_show = false
        this.$router.push({
          path: 'Maws',
          query: {
            placement: this.$route.query.placement,
            storage: this.$route.query.storage,
            location: this.$route.query.location,
            manager: this.$route.query.manager,
            date: this.$route.query.date,
          }
        });
      }

    }
  }

}
</script>
