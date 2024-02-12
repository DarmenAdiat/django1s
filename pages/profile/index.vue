<template>
  <v-container class='profile__container'>
    <v-row>

      <v-col lg='4' md='6' sm='12' style='min-width: 427px;max-width: 427px;'>
        <div class='profile__block'>
            <div class='profile__avatar'>
              <v-img :src='userImage' style="border-radius: 50%;height: 200px;width: 200px;"></v-img>
            </div>
            <div class='profile__text'>
              <h4>{{ user.last_name }} {{ user.first_name }} {{ user.third_name }}</h4>
              <h4>ИИН: {{ user.id_num }}</h4>
              <h4>Телефон: {{ user.phone }}</h4>
              <h4>Личная почта: {{ user.email }}</h4>
              <h4>Корпоративная почта: {{ user.email_corp }}</h4>
            </div>
          <v-btn
            class='button_hover_red'
            style='bottom:20px;position: absolute;margin: 0 20px; width: 85%;'
            to='profile/edit'
          >
            <v-icon
              left
            >
              settings
            </v-icon>Редактировать профиль</v-btn>
        </div>
      </v-col>

      <v-col lg='4' md='6' sm='12' style='min-width: 427px;max-width: 427px;'>
        <div class='profile__block'>
        </div>
      </v-col>
      <v-col lg='4' md='6' sm='12' style='min-width: 427px;max-width: 427px;'>
        <div class='profile__block'>
        </div>
      </v-col>


    </v-row>

    <div style='display: none'>
      <v-row>
        <h1>Профиль пользователя</h1><br>
        <h4 class="subheading">Логин: {{ user.username }}</h4>
        <h4 class="subheading">Фамилия: {{ user.last_name }}</h4>
        <h4 class="subheading">Имя: {{ user.first_name }}</h4>
        <h4 class="subheading">Отчество: {{ user.third_name }}</h4>
        <h4 class="subheading">Личная почта: {{ user.email }}</h4>
        <h4 class="subheading">Корпоративная почта: {{ user.email_corp }}</h4>
        <h4 class="subheading">Телефон: {{ user.phone }}</h4>
        <h4 class="subheading">ИИН: {{ user.id_num }}</h4>
      </v-row>
    </div>
  </v-container>
</template>
<!-- Google tag (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-QWEHCJ0LCR"></script>
<script>
window.dataLayer = window.dataLayer || [];
function gtag(){dataLayer.push(arguments);}
gtag('js', new Date());

gtag('config', 'G-QWEHCJ0LCR');
console.log('salk')
</script>
<script>
export default {
  name: "Profile",
  middleware: 'auth',
  data: () => ({
    // userImage: require('assets/profiles/person.png'),
  }),
  head: {
    title: 'Профиль пользователя',
  },
  computed: {
    userImage(){
      if (!this.$auth.user.profile_image) {
        return require('assets/profiles/person.png')
      } else {
        return this.$auth.user.profile_image
      }
    },
    user() {
      return this.$auth.user
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
