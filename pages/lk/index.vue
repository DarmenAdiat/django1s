<template>
  <v-container>
    <h1>Личный кабинет</h1>
    <h4 class="subheading">Логин: {{ user.username }}</h4>
    <h4 class="subheading">Имя: {{ user.first_name }}</h4>
    <h4 class="subheading">Фамилия: {{ user.last_name }}</h4>
    <h4 class="subheading">E-mail: {{ user.email }}</h4>
    <h4 class="subheading">Организация: {{ user.full_name }}</h4>
    <h4 class="subheading">ИИН: {{ user.id_num }}</h4>
    <h4 class="subheading">Вид пользователя: {{ user.status }}</h4>
    <h4 class="subheading">Доступные дни отпуска: {{ user.available_vacations }}</h4>
  </v-container>
</template>

<script>
export default {
  name: "lk",
  middleware: 'auth',

  data() {
    return {  }
  },
  computed: {
    user() {
      // return this.$store.state.auth.user
      return this.$auth.user
    }
  },

}
</script>

<style scoped>

</style>
