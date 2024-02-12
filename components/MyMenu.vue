<template>
  <v-app-bar
    app
  >
    <div class="brand mr-5">
      <v-img
        :src="require('@/assets/logo.png')" alt="logo"
      ></v-img>
    </div>
    <v-tabs
      color="grey darken-1"
    >
      <v-tab
        v-for="link in links"
        :key="link.title"
        :to="link.to"
      >
        {{ link.title }}
      </v-tab>
      <v-tab
        v-if="$auth.loggedIn"
        key="logout"
        @click="logout"
      >
        Выйти
      </v-tab>
    </v-tabs>
    </v-app-bar>
</template>

<script>

export default {
  name: "MyMenu",
  computed: {
    user() {
      return this.$auth.user.id_num
    },
    links() {
      if (!this.$auth.loggedIn) {
        return [
          {title: 'Главная', to: '/'},
          {title: 'Вйти', to: '/login'},
          {title: 'Регистрация', to: '/register'},
        ]
      } else {
        return [
          {title: 'Главная', to: '/'},
          // {title: 'Таблица', to: '/price'},
          {title: 'Заявки', to: '/loglist'},
          {title: 'Заявка на отпуск', to: '/vacation'},
          {title: 'Кабинет', to: '/profile'},
        ]
      }
    }
  },
  methods: {
    async logout() {
      const logoutData = { 'refresh': this.$auth.strategy.refreshToken.get() }
      await this.$auth.logout( { data: logoutData })
    }
  }
}
</script>

<style scoped>

</style>
