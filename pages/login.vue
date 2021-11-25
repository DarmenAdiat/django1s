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
        <v-form
          ref="form"
          v-model="valid"
        >
          <v-text-field
            v-model="login"
            :rules="loginRules"
            label="Логин"
            prepend-icon="mdi-account"
            counter
            required
          ></v-text-field>
          <v-text-field
            v-model="password"
            :rules="passwordRules"
            type="password"
            prepend-icon="mdi-lock"
            counter
            label="Пароль"
            required
          ></v-text-field>

          <div
            class="md-3"
          ></div>

          <v-btn
            class="mr-4"
            @click="submit"
          >
            Войти
          </v-btn>
          <v-btn @click="clear">
            Очистить
          </v-btn>
        </v-form>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>
export default {
  name: "Login",
  data: () => ({
    valid: false,
    login: '',
    password: '',
    loginRules: [
      v => !!v || 'Логин обязательное поле',
      // v => (v && v.length >= 10) || 'Минимум 10 символов',
    ],
    passwordRules: [
      v => !!v || 'Пароль обязательное поле',
    ],
  }),
  methods: {
    async submit() {
      if (this.$refs.form.validate()) {
        try {
          const login = {
            'username': this.login,
            'password': this.password
          }
          await this.$auth.loginWith('local', {data: login})
        } catch (err) {
          console.log(err)
        }
      }
    },
    clear() {
      this.$refs.form.reset()
    },
  },
}
</script>

<style scoped>

</style>
