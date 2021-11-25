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
        <h3>Заявление на отпуск</h3>
        <h4>Сотрудник: {{ user.first_name }} {{ user.last_name }}</h4>
        <h5 class="mt-1">Сегодня: {{ dateToday }}</h5>
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


          <v-row style="margin-top: 40px">
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
          </v-row>


          <v-row>
            <v-col>
            <v-date-picker
              v-model="date"
              :first-day-of-week="1"
              :allowed-dates="disablePastDates"
              locale="ru-ru"
              color="green"
              no-title
              range
              scrollable
            ></v-date-picker>
            </v-col>
            <v-col>
              <v-text-field
                v-model="availableVac"
                style="color: black; font-size: 24px;"
                label="Доступные дни отпуска"
                outlined
                readonly
              ></v-text-field>
            </v-col>
          </v-row>

          <v-spacer class="mt-5"></v-spacer>

          <v-btn
            width="100%"
            depressed
            color="primary"
            @click="submit"
          >
            Отправить заявку
          </v-btn>

          <v-dialog
            v-model="dialog"
            hide-overlay
            persistent
            width="300"
          >
            <v-card
              color="green"
              dark
            >
              <v-card-text style="text-align:center;padding: 24px 10px;">
                Заявление на отпуск отправляется
                <v-progress-linear
                  indeterminate
                  color="white"
                  style="margin-top: 5px;height:10px;"
                  class="mb-0"
                ></v-progress-linear>
              </v-card-text>
            </v-card>
          </v-dialog>
          <v-dialog
            v-model="dialog_error"
            hide-overlay
            persistent
            width="300"
          >
            <v-card
              color="red"
              dark
            >
              <v-card-text style="text-align:center;padding: 24px 10px;color: black">
                Ошибка отправки, попробуйте еще раз!
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
  loadingButton: false,

  data: vm => ({
    newMethods: [],
    valid: false,
    dialog: false,
    dialog_error: false,
    id_num: '', // ИИН
    vacationType: 0,// переменная со значением типа отпуска, значения от 1 до 8, integer
    availableVac: null, // заглушка для доступных дней отпуска
    doc_status: 2, // первоначальный статус документа
    doc_name: 'Заявка на отпуск', // название документа в бд

    days_used: 0, // количество выбранных дней отпуска - COUNT
    // КОЛИЧЕСТВО КАЛЕНДАРНЫХ ДНЕЙ ОТПУСКА !!! (не учитываются праздничные дни )

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
  }),

  computed: {
    // возвращаем все данные о текущем пользователе
    user() {
      return this.$auth.user
    },
  },

  beforeMount(){
    // this.loadingButton = true
    this.refreshVacation()
    // this.loadingButton = false
  },
  // refreshVacation()
  methods: {
    // отправление формы с данными заявления на API как в БД так и в 1С
    async submit() {
      if (!this.$refs.form.validate()) {
        // alert("ALARM! RABBITS ARE <!NOT!> FUCKING!"); // Выводим ошибку о том что не выбран тип отпуска
        return;
      }

      // генерируем идентификатор заявки в формате uuid4
      const idUUID = uuidv4();

      // проверка введенных дат
      if (!this.date) {
        alert("Выберите дату начала и окончания отпуска!"); // Выводим ошибку о  том что не выбраны дни отпуска
        return;
      }
      else if ( !this.date[1] ) {
        this.date[1] = this.date[0] // если выбран всего один день, то в обе ячейки ставим одну дату
      }

      try {
        this.dialog = true; // запускаем лоадер отправки формы

        // ---------------------------------------------
        // Добавить возможность ввода дат отпуска вручную !!!!!
        // Более понятно объянить ошибку при ошибке в 1С !!!!!!!!!
        // Округлить дни отпуска в большую сторону
        // Лоадер
        // ---------------------------------------------

        //  http://nls-test/testaman/hs/PrettyAPI/V2 - актуальный адрес для отправки запроса в 1С
        // КОД ДЛЯ ОТПРАВКИ ФОРМЫ В 1С [ ИИН, ИД-пользователя?, ИД-заявкиБД, датаНачалаОтпуска, датаОкончанияОтпуска, типОтпуска ]
        await this.$axios.$post('http://nls-test/testaman/hs/PrettyAPI/V2', {
          id: idUUID, // base64 идентификатор заявки (уникальный ключ)
          id_user: this.user.id,
          id_num: this.user.id_num,
          start_vacation: this.date.sort()['0'],
          end_vacation: this.date.sort()['1'],
          vacation_type: this.vacationType, // тип отпуска
        });

        // отправка заявки в БД таблицу заявок на отпуск
        await this.$axios.$post('api/v1/vacations/', {
          id: idUUID, // base64 идентификатор заявки (уникальный ключ)
          doc_name: this.doc_name,
          id_user: this.user.id,
          id_num: this.user.id_num,
          first_name: this.user.first_name,
          last_name: this.user.last_name,
          full_name: this.user.full_name,
          start_vacation: this.date.sort()['0'],
          end_vacation: this.date.sort()['1'],
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
    disablePastDates(val) {
      return val > new Date().toISOString().substr(0, 10)
    },

    // функция перезагрузки страницы
    refresh(){
      window.location.reload()
    },

    // ОБНОВЛЕНИЕ КОЛИЧЕСТВА ДОСТУПНЫХ ДНЕЙ ОТПУСКА
    async refreshVacation() {
      // alert(this.disablePastDates())
      this.loadingButton = true
      const availVac = await this.$axios.$get('http://nls-test/testaman/hs/PrettyAPI/V1', {
        params: {
          iin_num: this.user.id_num // отправляем в гет запросе ИИН авторизованного пользователя
        }
      })
      this.availableVac = Math.ceil(parseFloat(availVac.replace(",", "."))) // получение из 1С числа доступных дней отпуска
      // преобразуем строку в флоат и округляем в большую сторону
      this.loadingButton = false
      return availVac // возвращаем полученное значение функции
    },

    // функция заглушка
    someAction(){
      alert("Форма отправлена");
    },
  },

}
</script>

<style scoped>

</style>
