<template>
  <v-container>
    <v-row
      justify="center"
    >
      <v-col
        cols="12"
        sm="8"
        md="6"
        lg="4"
      >

      </v-col>
    </v-row>
    <v-row
      justify="center"
      style='margin-bottom: 50px;'
    >
      <v-col
        cols="12"
        sm="12"
        md="8"
        lg="6"
      >
        <h3>Регистрация пользователя</h3>
        <h5
          :class='message_iin'
        >{{message_info}}</h5>
        <v-row>
          <v-col cols='8' style='margin:0;padding: 10px;'>
            <v-text-field
              v-model="login"
              :rules="loginRules"
              label="ИИН"
              prepend-icon="mdi-account"
              :disabled='isDisabled_iin'
              counter
              required
            ></v-text-field>
          </v-col>
          <v-col cols='4' style='margin:0;padding: 10px;'>
            <v-btn
              class="mr-4 button_hover_red"
              style='margin-top:20px;width: 100%;font-size: 12px;'
              :disabled='isDisabled_iin'
              @click="checkIin"
            >
              Проверить ИИН
            </v-btn>
          </v-col>
        </v-row>

        <v-row v-if="visible_phone">
          <v-col cols='8' style='margin:0;padding: 10px;'>
            <v-text-field
              v-model="phone"
              v-mask="'+# (###) ###-####'"
              :rules="phoneRules"
              type='tel'
              label="Номер телефона"
              prepend-icon="mdi-phone"
              :disabled='isDisabled_phone'
              placeholder='+7 (705) 123-4567'
              counter
              required
            ></v-text-field>
          </v-col>
          <v-col cols='4' style='margin:0;padding: 10px;'>
            <v-btn
              class="mr-4 button_hover_red"
              style='margin-top:20px;width: 100%;font-size: 10px;'
              :disabled='isDisabled_phone'
              @click="enterPhone"
            >
              Привязать номер
            </v-btn>
          </v-col>
          <h5 v-if="visible_phone_note" style='color:#FF5722'><v-icon style='color:#FF5722'>mdi-exclamation-thick</v-icon>Номер телефона должен быть связан с аккаунтом Telegram</h5>
        </v-row>

        <v-row v-if="visible">
          <v-btn
            class="mr-4 button_hover_red"
            style='margin-top:20px;width: 100%'
            href="https://t.me/Isker_Group_Bot" target="_blank"
            :disabled='isDisabled_oneoff'
          >
            Получить одноразовый пароль у Telegram-бота
          </v-btn>
        </v-row>
        <v-row
          v-if="visible"
          style='margin:40px 0 0 0;'
        >
          <h5
            :class='message_pass'
          >{{message_info_pass}}</h5>
        </v-row>
        <v-row
          v-if="visible"
          class='show_pass'
          style='margin-top:0px;'
        >
          <v-col cols='8' style='margin:0;padding: 10px;'>
            <v-text-field
              v-model="one_off"
              :rules="passwordRulesOneOff"
              :append-icon="show ? 'mdi-eye' : 'mdi-eye-off'"
              :type="show ? 'text' : 'password'"
              prepend-icon="mdi-lock"
              :disabled='isDisabled_oneoff'
              counter
              label="Одноразовый пароль"
              required
              @click:append="show = !show"
            ></v-text-field>
          </v-col>
          <v-col cols='4' style='margin:0;padding: 10px;'>
            <v-btn
              class="mr-4 button_hover_red"
              style='margin-top:20px;width: 100%'
              :disabled='isDisabled_oneoff'
              @click="checkPass"
            >
              Вход
            </v-btn>
          </v-col>
        </v-row>
        <v-row
          v-if="visible_register"
          class='show_register'
          style='margin:50px 0 0;'
        >
          <v-form
            ref="form"
            v-model="valid"
            style='width: 100%'
          >
            <h3>Установите свой пароль</h3>
            <v-text-field
              v-model="password"
              :rules="passwordRules"
              :append-icon="show2 ? 'mdi-eye' : 'mdi-eye-off'"
              :type="show2 ? 'text' : 'password'"
              prepend-icon="mdi-lock"
              counter
              label="Введите новый пароль"
              required
              @click:append="show2 = !show2"
            ></v-text-field>
            <v-text-field
              v-model="passwordRepeat"
              :rules="[passwordConfirmationRule]"
              :append-icon="show3 ? 'mdi-eye' : 'mdi-eye-off'"
              :type="show3 ? 'text' : 'password'"
              prepend-icon="mdi-lock"
              counter
              label="Повторите новый пароль"
              required
              @click:append="show3 = !show3"
            ></v-text-field>
            <h3>Введите актуальные данные</h3>

            <v-text-field
              v-model="email"
              label="Личная почта"
              prepend-icon="mdi-email"
              counter
            ></v-text-field>
            <v-text-field
              v-model="email_corp"
              :rules="emailRules"
              label="Корпоративная почта"
              prepend-icon="mdi-email"
              counter
              required
            ></v-text-field>

            <v-checkbox
              v-model="checkbox"
              :rules="checkboxRules"
            >
              <template #label>
                <div>
                  Даю свое
                  <v-tooltip bottom>
                    <template #activator="{ on }">
                      <a
                        target="_blank"
                        href="/privacy.docx"
                        @click.stop
                        v-on="on"
                      >
                        Согласие
                      </a>
                    </template>
                    Согласие на обработку персональных данных
                  </v-tooltip>
                  на обработку персональных данных
                </div>
              </template>
            </v-checkbox>


            <v-btn
              class="mr-4 button_hover_red"
              style='margin-top:20px'
              @click="submit"
            >
              Сохранить и войти
            </v-btn>
          </v-form>
        </v-row>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>

import { VueMaskDirective } from 'v-mask'
import Vue from 'vue'
Vue.directive('mask', VueMaskDirective);

export default {
  name: "Register",

  data: () => ({
    show: false,
    show2: false,
    show3: false,

    valid: false,
    isDisabled_iin: false,
    isDisabled_oneoff: false,
    isDisabled_phone: false,
    login: '',
    one_off: '',
    phone: '',
    email: '',
    email_corp: '',
    checkbox: false,
    visible: false,
    visible_phone: false,
    visible_register: false, // ========================= uncomment the
    visible_phone_note: true,

    // visible: true,
    // visible_phone: true,
    // visible_register: true, // ========================= uncomment the

    checker: [],
    password: '',
    passwordRepeat: '',
    message_iin: 'message_info',
    message_info: 'Введите ваш ИИН',
    message_color: 'mediumseagreen',

    message_pass: 'message_info',
    message_info_pass: 'Введите одноразовый пароль',

    loginRules: [
      v => !!v || 'ИИН обязательное поле',
      v => v && v.length === 12 || "ИИН состоит из 12-ти цифр",
      v => !v || /[0-9]/.test(v) || 'ИИН должен состоять только из цифр',
    ],
    passwordRulesOneOff: [
      v => !!v || 'Это обязательное поле',
      v => v && v.length <= 100 || "Максимум 100 символов",
    ],
    passwordRules: [
      v => !!v || 'Пароль обязательное поле',
      v => v && v.length <= 100 || "Максимум 100 символов",
      v => v && v.length >= 8 || "Минимум 8 символов",
      v => !v || /\d/.test(v) || 'Пароль должен содержать минимум одну цифру',
      v => !v || /[a-z]/.test(v) || 'Пароль должен содержать минимум одну букву нижнего регистра',
      // v => !v || /[A-Z]/.test(v) || 'Пароль должен содержать минимум одну букву верхнего регистра',
      // v => !v || /[!@#\$%\^\&*\)\(+=._-]/.test(v) || 'Пароль содержит недопустимые символы',
      // v => !v || /[A-Za-z0-9!@#$%^&*)(+=._-]/.test(v) || 'wrong_char' ,
      // v => !v || /[A-Za-z0-9\!@#\$%\^\&*\)\(+=._-]/.test(v) || 'Пароль содержит недопустимые символы',
    ],
    phoneRules: [
      v => !!v || 'Телефон обязательное поле',
      v => v && v.length === 17 || "Телефон обязательное поле",
    ],
    emailRules: [
      v => !!v || 'Email обязательное поле',
      v => v && v.length <= 100 || "Максимум 100 символов",
      v => !v || /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v) || 'E-mail введен неправильно'
    ],
    checkboxRules:[
      v => v === true || 'Подтвердите свое согласие',
    ],

  }),
  head: {
    title: 'Регистрация',
  },
  computed: {
    passwordConfirmationRule() {
      return () => (this.password === this.passwordRepeat) || 'Пароли не совпадают'
    },
  },
  methods: {
    async submit() {
      if (this.$refs.form.validate()) {
        const generated = this.generatePass()
        try {
          await this.$axios.$patch('api/v1/users/' + this.checker.id + '/', {
            password: this.password,
            is_active: "True",
            one_off: generated,
            email: this.email,
            email_corp: this.email_corp,
          });

          await new Promise(resolve => this.$router.push('/login'));
        } catch (e) {
          console.log('error, not sent')
        }
      }
    },

    async checkIin(){

      try {
        this.login = this.login.replace(/[^0-9]/g, '')
        // console.log(this.login.length)
        // this.login = parseInt(this.login)
        if (this.login.length === 12) {
          console.log('IIN OK')
        } else {
          this.message_info = 'ИИН должен состоять из 12-ти цифр'
          this.message_iin = 'message_info_error'
          this.login = ''
          console.log("Not ok")
          return
        }

        const response = await this.$axios.$get('api/v1/check/'+ this.login +'/')
        this.checker.id = response[0].id
        this.checker.id_num = response[0].id_num
        this.checker.is_active = response[0].is_active
        this.checker.one_off = response[0].one_off

        if (this.checker.is_active === true) {
          this.message_info = 'Аккаунт с этим ИИН уже активирован'
          this.message_iin = 'message_info_error'
          return
        } else {
          this.message_info = 'Введенный вами ИИН доступен для активации'
          this.message_iin = 'message_info_success'
          this.isDisabled_iin = true
          this.visible_phone = true
        }

        return this.checker
      } catch (e) {
        console.log('Введенный вами ИИН не найден')
        this.message_info = 'Введенный вами ИИН не найден'
        this.message_iin = 'message_info_error'
      }
    },

    async checkPass(){
      try {
        const response = await this.$axios.$get('api/v1/check/'+ this.login +'/')
        this.checker.id = response[0].id
        this.checker.id_num = response[0].id_num
        this.checker.is_active = response[0].is_active
        this.checker.one_off = response[0].one_off

        if (this.checker.is_active === true) {
          this.message_info = 'Аккаунт с этим ИИН уже активирован'
          this.message_iin = 'message_info_error'
          return
        } else {
          this.message_info = 'Введенный вами ИИН доступен для активации'
          this.message_iin = 'message_info_success'
          this.isDisabled_iin = true
        }
        // ======================= password one off check
        if (this.one_off === this.checker.one_off) {
          this.message_info_pass = 'Одноразовый пароль принят'
          this.message_pass = 'message_info_success'
          this.isDisabled_oneoff = true
          this.visible_register = true
        } else {
          this.message_info_pass = 'Введенный вами пароль неверный'
          this.message_pass = 'message_info_error'
        }
        // ======================= password one off check
        return this.checker
      } catch (e) {
        console.log('Введенный вами ИИН не найден')
        this.message_info = 'Введенный вами ИИН не найден'
        this.message_iin = 'message_info_error'
      }





    },

    async enterPhone(){
      let sendPhone = this.phone.replace(/[^0-9]/g, '')
      sendPhone = '+'+sendPhone

      try {
        await this.$axios.$patch('api/v1/users/' + this.checker.id + '/', {
          phone: sendPhone,
        });
        this.isDisabled_phone = true
        this.visible = true
        this.visible_phone_note = false // скрываем текст о том что телефон должен быть с Телеграм-акком
        console.log('Sent: '+sendPhone)
      } catch (e) {
        console.log('error, phone not sent')
      }
    },

    generatePass() {
      const generatePass = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
      let password = '';

      for(let i=0; i < 15; i++) {
        password += generatePass.charAt(Math.floor(Math.random() * generatePass.length));
      }
      return password
    },

  },
}
</script>

<style scoped>
.button_hover_red:hover{
  color:#fff!important;
  background-color: #FF6B6B!important;
}
</style>
