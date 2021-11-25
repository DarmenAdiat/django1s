<template>
  <v-container>
    <v-row
      justify="center"
      class="mt-10"
    >
      <v-col
        cols="12"
        sm="8"
        md="6"
        lg="4"
      >
        <h1>Заявление на отпуск</h1>
        <h3>Сотрудник: {{ user.first_name }} {{ user.last_name }}</h3>
        <h4 class="mt-1">Сегодня: {{ dateToday }}</h4>
        <v-spacer class="mt-5"></v-spacer>
        <v-form
          ref="form"
          v-model="valid"
          @submit.prevent="someAction()"
        >
          <v-text-field
            v-model="user.first_name"
            label="Имя"
            :rules="fieldRules"
            disabled
            required
            style="display: none"
          ></v-text-field>
          <v-text-field
            v-model="user.last_name"
            label="Фамилия"
            :rules="fieldRules"
            disabled
            required
            style="display: none"
          ></v-text-field>
          <v-text-field
            v-model="user.id_num"
            label="ИИН"
            :rules="fieldRules"
            disabled
            required
            style="display: none"
          ></v-text-field>
          <v-text-field
            v-model="user.full_name"
            label="Компания"
            :rules="fieldRules"
            disabled
            required
            style="display: none"
          ></v-text-field>
          <v-text-field
            v-model="doc_status"
            label="Статус"
            :rules="fieldRules"
            disabled
            style="display: none"
          ></v-text-field>
          <v-text-field
            v-model="doc_name"
            label="Название документа"
            :rules="fieldRules"
            disabled
            style="display: none"
          ></v-text-field>
          <v-text-field
            v-model="user.id"
            label="Идентификатор пользователя"
            :rules="fieldRules"
            disabled
            style="display: none"
          ></v-text-field>

          <v-row style="margin-top: 20px">
            <v-col>
              <v-select
                v-model="vacationType"
                :items="items"
                item-text="name"
                filled
                label="Выберите тип отпуска"
                :rules="fieldRules"
                required
                outlined
              ></v-select>
            </v-col>
          </v-row>

          <v-row>
            <v-col style="border: 2px solid lightgray; border-right: 1px solid lightgray; border-top-left-radius: 10px;border-bottom-left-radius: 10px;">
              <h3 style="margin-bottom: 20px;text-align: center;">Выберите период отпуска</h3>
              <v-row style="margin: 0;">
              <v-menu
                ref="menu1"
                v-model="menu1"
                :close-on-content-click="false"
                transition="scale-transition"
                offset-y
                max-width="290px"
                min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                    v-model="dateFormatted1"
                    label="Дата начала отпуска"
                    outlined
                    prepend-icon="mdi-calendar"
                    v-bind="attrs"
                    @blur="date1 = parseDate1(dateFormatted1)"
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="date1"
                  :first-day-of-week="1"
                  :allowed-dates="disablePastDates1"
                  locale="ru-ru"
                  color="green"
                  no-title
                  scrollable
                  @input="menu1 = false"
                ></v-date-picker>
              </v-menu>
            </v-row>
              <v-row style="margin: 0;">
              <v-menu
                ref="menu2"
                v-model="menu2"
                :close-on-content-click="false"
                transition="scale-transition"
                offset-y
                max-width="290px"
                min-width="auto"
              >
                <template v-slot:activator="{ on, attrs }">
                  <v-text-field
                    v-model="dateFormatted2"
                    label="Дата окончания отпуска"
                    outlined
                    prepend-icon="mdi-calendar"
                    v-bind="attrs"
                    @blur="date2 = parseDate2(dateFormatted2)"
                    v-on="on"
                  ></v-text-field>
                </template>
                <v-date-picker
                  v-model="date2"
                  :first-day-of-week="1"
                  :allowed-dates="disablePastDates2"
                  locale="ru-ru"
                  color="green"
                  no-title
                  scrollable
                  @input="menu2 = false"
                ></v-date-picker>
              </v-menu>
            </v-row>
            </v-col>
            <v-col style="border: 2px solid lightgray; border-left: 1px solid lightgray; border-top-right-radius: 10px;border-bottom-right-radius: 10px;">
              <h3 style="margin-bottom: 20px;text-align: center;">Доступные дни отпуска</h3>
              <v-text-field
                v-model="availableVac"
                :loading="loadingField ? 'orange accent-4' : null"
                :loader-height="7"
                style="color: black;font-size: 24px;"
                outlined
                readonly
              ></v-text-field>
            </v-col>
          </v-row>

          <v-row style="margin-top: 20px;">
            <v-col>
              <v-btn
                width="100%"
                height="48"
                depressed
                color="primary"
                @click="submit"
              >
                Отправить заявку
              </v-btn>
            </v-col>
          </v-row>

          <v-dialog
            v-model="dialog"
            hide-overlay
            persistent
            width="400"
          >
            <v-card
              color="green"
              dark
            >
              <v-card-text style="text-align:center;padding: 24px 10px;">
                <h2 style="margin:20px;color: white;">Ваше заявление на отпуск<br>отправляется...</h2>
                <v-progress-linear
                  indeterminate
                  color="white"
                  style="margin: 20px 5px 20px 5px; height:10px; width: auto;"
                ></v-progress-linear>
              </v-card-text>
            </v-card>
          </v-dialog>
          <v-dialog
            v-model="dialog_error"
            hide-overlay
            persistent
            width="400"
          >
            <v-card
              color="red"
              dark
            >
              <v-card-text style="text-align:center;padding: 24px 20px;color: white;">
                <h2 style="margin:5px;">Ошибка на сервере 1С</h2>
                <h3 style="margin:5px;">Ваше заявление на отпуск не отправлено</h3>
                <h3 style="margin:20px;">Попробуйте еще раз<br>или обратитесь в службу поддержки</h3>
                <v-btn
                  width="100%"
                  depressed
                  color="black"
                  @click="refresh"
                >
                  Перезагрузить страницу
                </v-btn>
              </v-card-text>
            </v-card>
          </v-dialog>
          <v-dialog
            v-model="dialog_validate"
            hide-overlay
            persistent
            width="400"
          >
            <v-card
              color="yellow"
              dark
            >
              <v-card-text style="text-align:center;padding: 24px 20px;color: black;">
                <h2 style="margin:5px;">Ошибка при заполнениее формы!</h2>
                <h3 style="margin:5px;">Ваше заявление на отпуск не отправлено</h3>
                <h3 style="margin:20px;">Вы неправильно выбрали<br>даты начала и окончания отпуска</h3>
                <v-btn
                  width="100%"
                  depressed
                  color="black"
                  @click="dialog_validate=false"
                >
                  Продолжить заполнение формы
                </v-btn>
              </v-card-text>
            </v-card>
          </v-dialog>
        </v-form>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
import moment from 'moment';

const { v4: uuidv4 } = require('uuid');
uuidv4();

export default {
  name: "vacation",
  components: {},
  middleware: 'auth',

  data: vm => ({
    newMethods: [],
    valid: false,
    dialog: false,
    dialog_error: false,
    dialog_validate: false,
    id_num: '', // ИИН
    vacationType: 0,// переменная со значением типа отпуска, значения от 1 до 8, integer
    availableVac: null, // заглушка для доступных дней отпуска
    doc_status: 2, // первоначальный статус документа
    doc_name: 'Заявка на отпуск', // название документа в бд
    days_used: 0, // количество выбранных дней отпуска - COUNT
    // КОЛИЧЕСТВО КАЛЕНДАРНЫХ ДНЕЙ ОТПУСКА !!! (не учитываются праздничные дни )
    loadingField: true,
    fieldRules: [
      (v) => !!v || 'Это обязательное поле',
    ],  // простое правило проверки полей формы перед отправкой

    // Массив объектов: Описание и значени типов отпуска
    items: [
      {
        name: 'Ежегодный оплачиваемый отпуск - 24 дня',
        value: 1,
      },
      {
        name: 'Отпуск без сохранения заработной платы - по соглашению',
        value: 2,
      },
      {
        name: 'Отпуск для регистрации брака - 3 дня',
        value: 3,
      },
      {
        name: 'Отпуск при рождении ребенка - 1 день',
        value: 4,
      },
      {
        name: 'Отпуск для подготовки к детскому саду - 14 дней',
        value: 5,
      },
      {
        name: 'Отпуск в День Знаний - 1 день',
        value: 6,
      },
      {
        name: 'Отпуск по семейным обстоятельствам - 5 дней',
        value: 7,
      },
      {
        name: 'Отпуск на медицинский осмотр - 1 день',
        value: 8,
      },
    ],

    date: null, // массив выбранных дат. Пример фомарата массива: ['2020-02-02', '2020-02-02']

    dateToday: moment(new Date()).format('DD/MM/yyyy'),

// ============================================================================
//     date1: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),

    date1: moment(new Date()).format('yyyy-MM-DD'),
    date2: moment(new Date()).format('yyyy-MM-DD'),

    dateFormatted1: vm.formatDate1((new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10)),
    // date2: (new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10),
    dateFormatted2: vm.formatDate2((new Date(Date.now() - (new Date()).getTimezoneOffset() * 60000)).toISOString().substr(0, 10)),
    menu1: false,
    menu2: false,
// ============================================================================
  }),

  computed: {
    // возвращаем все данные о текущем пользователе
    user() {
      return this.$auth.user
    },
  },

  watch: {
    date1 (val) {
      this.dateFormatted1 = this.formatDate1(this.date1)
    },
    date2 (val) {
      this.dateFormatted2 = this.formatDate2(this.date2)
    },
  },

  beforeMount(){
    this.refreshVacation()
  },

  methods: {
    // отправление формы с данными заявления на API как в БД так и в 1С
    async submit() {
      if (!this.$refs.form.validate()) {
        // ПОСТАВИТЬ ОКНО ОШИБКИ ВМЕСТО АЛЕРТа для "детей" с подтверждением
        // alert("ALARM! RABBITS ARE <!NOT!> FUCKING!"); // Выводим ошибку о том что не выбран тип отпуска
        return;
      }
      if ( moment(this.date1).isAfter(this.date2) ) {
        // ПОСТАВИТЬ ОКНО ОШИБКИ ВМЕСТО АЛЕРТа для "детей" с подтверждением
        this.dialog_validate = true
        // alert("Дата окончания отпуска должна быть не меньше даты начала"); // Выводим ошибку о  том что не выбраны дни отпуска
        return;
      }


      // генерируем идентификатор заявки в формате uuid4
      const idUUID = uuidv4();

      // проверка введенных дат
      if (!this.date1) {
        alert("Выберите дату начала отпуска!"); // Выводим ошибку о  том что не выбраны дни отпуска
        return;
      }
      if (!this.date2) {
        alert("Выберите дату окончания отпуска!"); // Выводим ошибку о  том что не выбраны дни отпуска
        return;
      }

      try {
        this.dialog = true; // запускаем лоадер отправки формы

        // ---------------------------------------------
        // Более понятно объянить ошибку при ошибке в 1С !!!!!!!!!
        // ---------------------------------------------

        //  http://nls-test/testaman/hs/PrettyAPI/V2 - актуальный адрес для отправки запроса в 1С
        // КОД ДЛЯ ОТПРАВКИ ФОРМЫ В 1С [ ИИН, ИД-пользователя?, ИД-заявкиБД, датаНачалаОтпуска, датаОкончанияОтпуска, типОтпуска ]
        // await this.$axios.$post('http://nls-test/testaman/hs/PrettyAPI/V2123', {
        //   id: idUUID, // base64 идентификатор заявки (уникальный ключ)
        //   id_user: this.user.id,
        //   id_num: this.user.id_num,
        //   start_vacation: this.date1,
        //   end_vacation: this.date2,
        //   vacation_type: this.vacationType, // тип отпуска
        // });

        // отправка заявки в БД таблицу заявок на отпуск
        await this.$axios.$post('api/v1/vacations123/', {
          id: idUUID, // base64 идентификатор заявки (уникальный ключ)
          doc_name: this.doc_name,
          id_user: this.user.id,
          id_num: this.user.id_num,
          first_name: this.user.first_name,
          last_name: this.user.last_name,
          full_name: this.user.full_name,
          start_vacation: this.date1,
          end_vacation: this.date2,
          vacation_status: this.doc_status,
          vacation_type: this.vacationType, // тип отпуска
        });
        //
        // отправка заявки в БД таблицу логирования
        await this.$axios.$post('api/v1/loglist/', {
          id: idUUID, // base64 идентификатор заявки (уникальный ключ)
          id_user: this.user.id,
          id_num: this.user.id_num,
          first_name: this.user.first_name,
          last_name: this.user.last_name,
          full_name: this.user.full_name,
          doc_name: this.doc_name,
          doc_status: this.doc_status,
        });

        // 2,5 секунды тайм-аута для вывода лоадера при отправке данных - визуальная инициализация отправки заявления
        // чтобы пользователь увидет что все отправилось и
        // ПОКАЗАТЬ ЧТОТО ТИПО ГАЛОЧКИ ПЕРЕД РЕДИРЕКТОМ
        await new Promise(resolve => setTimeout(() => { (this.dialog = false); (this.$router.push('/loglist')); }, 2500));
        // await new Promise(resolve => setTimeout(() => { (this.dialog = false); (alert("HI WATAFAKER 2")); }, 2500));
        // await this.$router.push('/loglist'); // перенаправление пользователя на страницу его заявок
        // this.dialog = false
        // (this.$router.push('/loglist'))
      } catch (e) {
        this.dialog_error = true // открывает диалоговое окно об ошибке с кнопкой перезагрузки
      }
    },

    // Отключение недоступных дат для выбора отпуска
    disablePastDates1(val) {
      return val > moment(new Date()).format('yyyy-MM-DD')
    },
    disablePastDates2(val) {

      const createdDate = moment(this.date1).subtract(1, 'day').format('yyyy-MM-DD')
      const lastDate = moment(this.date1).add(this.availableVac, 'day').format('yyyy-MM-DD')

      const val2 = createdDate
      const val3 = lastDate

      return val > val2 && val < val3
    },

    // функция перезагрузки страницы
    refresh(){
      window.location.reload()
    },

    // ОБНОВЛЕНИЕ КОЛИЧЕСТВА ДОСТУПНЫХ ДНЕЙ ОТПУСКА
    async refreshVacation() {
      this.loadingField = true
      try {
        const availVac = await this.$axios.$get('http://nls-test/testaman/hs/PrettyAPI/V1', {
          params: {
            iin_num: this.user.id_num // отправляем в гет запросе ИИН авторизованного пользователя
          }
        })
        this.availableVac = Math.ceil(parseFloat(availVac.replace(",", "."))) // получение из 1С числа доступных дней отпуска
        // преобразуем строку в флоат и округляем в большую сторону
        this.loadingField = false
        return availVac // возвращаем полученное значение функции
      } catch (e) {
        this.dialog_error = true // открывает диалоговое окно об ошибке с кнопкой перезагрузки
      }
    },

    // функция заглушка
    someAction(){
      alert("Форма отправлена");
    },

    // date1 и date2 преобразовываются после выбора даты в формат - "2020-01-30"
    formatDate1 (date1) {
      if (!date1) return null

      const [year1, month1, day1] = date1.split('-')
      return `${day1}/${month1}/${year1}`
    },
    parseDate1 (date1) {
      if (!date1) return null

      const [day1, month1, year1] = date1.split('/')
      return `${year1}-${month1.padStart(2, '0')}-${day1.padStart(2, '0')}`
    },
    formatDate2 (date2) {
      if (!date2) return null

      const [year2, month2, day2] = date2.split('-')
      return `${day2}/${month2}/${year2}`
    },
    parseDate2 (date2) {
      if (!date2) return null

      const [day2, month2, year2] = date2.split('/')
      return `${year2}-${month2.padStart(2, '0')}-${day2.padStart(2, '0')}`
    },

  },

}
</script>

<style scoped>

</style>
