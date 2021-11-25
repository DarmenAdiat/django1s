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
    links() {
      if (!this.$auth.loggedIn) {
        return [
          {title: 'Главная', to: '/'},
          {title: 'Войти', to: '/login'},
        ]
      } else {
        return [
          {title: 'Главная', to: '/'},
          // {title: 'Таблица', to: '/price'},
          {title: 'Заявки', to: '/loglist'},
          {title: 'Заявка на отпуск', to: '/vacation'},
          {title: 'Кабинет', to: '/lk'},
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
