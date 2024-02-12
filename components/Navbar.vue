<template>
  <nav>
    <v-app-bar color='#F7FFF7' dark app>
      <v-app-bar-nav-icon class='button_hover_red_burger' style='color:#1a535cff;' title='Свернуть/развернуть боковое меню' @click.stop='drawer = !drawer'>

      </v-app-bar-nav-icon>
      <v-toolbar-title style='cursor: pointer;' title='Перейти в панель управления'>
        <a href='/' style='text-decoration: none;color:#1a535cff;'>
          <span>ISKER </span>
          <span class='font-weight-light'>Group</span>
        </a>
      </v-toolbar-title>
      <v-spacer></v-spacer>
      <v-btn
        v-if="admin" offset-y
        href='http://45.86.80.42:8000/admin'
        target="_blank"
        text
        exact
        class="primary--text"
      >
        Админка
      </v-btn>
<!--      <LocaleChanger/>-->
      <v-btn
        v-if='$auth.loggedIn'
        key='logout'
        class='button_hover_red'
        title='Выйти из аккаунта'
        text
        @click='logout'
      >
        <span>Выйти</span>
        <v-icon right> exit_to_app</v-icon>
      </v-btn>
    </v-app-bar>
    <v-navigation-drawer v-model='drawer' dark app style='background-color: #1a535cff'>
      <v-layout column align-center>
        <v-flex v-if='$auth.loggedIn' class='mt-5' style='text-align: center'>
          <v-avatar size='100'>
            <v-img :src='userImage'></v-img>
          </v-avatar>
          <p class='white--text subheading mt-3 text-center'>{{ user.first_name }} {{ user.last_name }}</p>
        </v-flex>
      </v-layout>
      <v-list id='flat' flat>
        <v-list-item v-for='link in links' :key='link.text' router :to='link.route' active-class='border'>
          <v-list-item-action>
            <v-icon>{{ link.icon }}</v-icon>
          </v-list-item-action>
          <v-list-item-content>
            <v-list-item-title>{{ link.text }}</v-list-item-title>
          </v-list-item-content>
        </v-list-item>


        <v-list-group
          v-if='$auth.loggedIn'
          prepend-icon='account_box'
        >
          <template #activator>
            <v-list-item-title>Личный кабинет</v-list-item-title>
          </template>
          <v-list-item-content style='padding: 0 0 0 20px;'>
            <v-list-item
              v-for='(admin, i) in account'
              :key='i'
              router
              :to='admin.route'
              active-class='border'
            >
              <v-list-item-icon>
                <v-icon v-text='admin.icon'></v-icon>
              </v-list-item-icon>
              <v-list-item-title v-text='admin.text'></v-list-item-title>
            </v-list-item>
          </v-list-item-content>
        </v-list-group>

        <v-list-group
          v-if='$auth.loggedIn'
          prepend-icon='devices'
        >
          <template #activator>
            <v-list-item-title>Основные средства</v-list-item-title>
          </template>
          <v-list-item-content style='padding: 0 0 0 20px;'>
            <v-list-item
              v-for='(admin, i) in os'
              :key='i'
              router
              :to='admin.route'
              active-class='border'
            >
              <v-list-item-icon>
                <v-icon v-text='admin.icon'></v-icon>
              </v-list-item-icon>
              <v-list-item-title v-text='admin.text'></v-list-item-title>
            </v-list-item>
          </v-list-item-content>
        </v-list-group>

        <v-list-group
          v-if='$auth.loggedIn'
          prepend-icon='store_mall_directory'
        >
          <template #activator>
            <v-list-item-title>Склад</v-list-item-title>
          </template>
          <v-list-item-content style='padding: 0 0 0 20px;'>
            <v-list-item
              v-for='(admin, i) in store'
              :key='i'
              router
              :to='admin.route'
              active-class='border'
            >
              <v-list-item-icon>
                <v-icon v-text='admin.icon'></v-icon>
              </v-list-item-icon>
              <v-list-item-title v-text='admin.text'></v-list-item-title>
            </v-list-item>
          </v-list-item-content>
        </v-list-group>

        <v-list-group
          v-if='$auth.loggedIn'
          prepend-icon='people'
        >
          <template #activator>
            <v-list-item-title>HR</v-list-item-title>
          </template>
          <v-list-item-content style='padding: 0 0 0 20px;'>
            <v-list-item
              v-for='(admin, i) in hr'
              :key='i'
              router
              :to='admin.route'
              active-class='border'
            >
              <v-list-item-icon>
                <v-icon v-text='admin.icon'></v-icon>
              </v-list-item-icon>
              <v-list-item-title v-text='admin.text'></v-list-item-title>
            </v-list-item>
          </v-list-item-content>
        </v-list-group>


      </v-list>
    </v-navigation-drawer>
  </nav>
</template>

<script>
import Vue from 'vue'
import VueGtag from 'vue-gtag'
// import LocaleChanger from '@/components/LocaleChanger'

export default {
  modules: [
    '@nuxtjs/gtm'
  ],
  gtm: {
    id: 'GTM-PNTNQ2L',
  },
  name: 'Navbar',
  // components: {
  //   LocaleChanger
  // },
  data: () => ({
    drawer: true,
    // userImage: require('assets/profiles/person.png'),
    // userImage: $auth.user.profile_image
    model: 'MENU'
  }),
  computed: {
    links() {
      if (!this.$auth.loggedIn) {
        return [
          { icon: 'mdi-login', text: 'Войти', route: '/login' },
          { icon: 'account_box', text: 'Регистрация', route: '/register' }
        ]
      } else {
        return [
          { icon: 'dashboard', text: 'Главная страница', route: '/' },
        ]
      }
    },
    admins() {
      return [
        { icon: 'view_in_ar', text: 'MAWS', route: '/Mawslist' }
      ]
    },
    admin() {
      return this.user !== null ? this.user.is_staff : false
    },
    hr() {
      return [
        { icon: 'flight', text: 'Отпуск', route: '/Vacationlist' }
      ]
    },
    store() {
      return [
        { icon: 'view_in_ar', text: 'MAWS', route: '/Mawslist' }
      ]
    },
    os(){
      const myArray = ['860411400564', '861004300558', '010731551241', '940419300162', '880315400615', '900325400517', '820129300012', '850920302321','810120350355'];
      const iin = this.$auth.user.id_num
      if(myArray.includes(iin)){
        return [
          { icon: 'list', text: 'Список ОС', route: '/OS'},
          { icon: 'list', text: 'Перемещение ОС', route: '/MoveOS'}
        ]
      }else{
        return [
          { icon: 'list', text: 'Список ОС', route: '/OS'}
        ]
      }

    },
    account() {
      return [
        { icon: 'account_circle', text: 'Профиль', route: '/profile' },
        { icon: 'attach_money', text: 'Payslip', route: '/payslip' },
        { icon: 'list', text: 'Все документы', route: '/loglist' },
      ]
    },
    user() {
      return this.$auth.user
    },
    UserForGoogle() {
      return {
        iin: this.$auth.user.id_num,
        full_name: this.$auth.user.last_name + " " + this.$auth.user.first_name,
      }
    },
    userImage(){
      if (!this.$auth.user.profile_image) {
        return require('assets/profiles/person.png')
      } else {
        return this.$auth.user.profile_image
      }
    }
  },

  // created() {
  //   this.initialize()
  // },
  mounted(){
    this.postGoogle()
  },
  methods: {
    sendToGoogle() {
      (function(w, d, s, l, i) {
          w[l] = w[l] || []
          w[l].push({
            'gtm.start':
              new Date().getTime(),
            event: 'gtm.js'
          })
          const f = d.getElementsByTagName(s)[0]
          const j = d.createElement(s)
          const dl = l !== 'dataLayer' ? '&l=' + l : ''
          j.async = true
          j.src = 'https://www.googletagmanager.com/gtm.js?id=' + i + dl
          f.parentNode.insertBefore(j, f)
        }
      )
      (window, document, 'script', 'dataLayer', 'GTM-PNTNQ2L');
    },
    postGoogle(){
      if(this.$auth.loggedIn){
        this.sendToGoogle()
      }
      window.dataLayer = window.dataLayer || [];
      window.dataLayer.push({
        'event': 'login',
        'user_idnum': this.UserForGoogle.iin,
        'full_name_id': this.UserForGoogle.full_name
      })
      Vue.use(VueGtag, {
        config: {
          id: 'G-QWEHCJ0LCR',
          event: 'login',
          // params:{
          // }
        }
      })
      return true
    },
    // initialize(){
    //   this.userImage = this.user.profile_image
    //   console.log(this.user.profile_image)
    //   console.log(this.userImage)
    // },
    async logout() {
      const logoutData = { 'refresh': this.$auth.strategy.refreshToken.get() }
      await this.$auth.logout({ data: logoutData })
        .then(this.$router.push('/login')) // перенаправление после логаута
    }
  }
}
</script>

<style scoped>
.border {
  /*border-left: 4px solid #ff575f;*/
  background-color: white;
  color: black;
}
#flat .v-list-item:hover{
  color: #FF6B6B!important;
}
#flat .v-list-item:hover .v-icon{
  color: #FF6B6B!important;
}



#flat .v-list-item--active{
  color: #fff!important;
  background-color: #FF6B6B!important;
}

#flat .v-list-item--active:hover{
  color: #fff!important;
}
#flat .v-list-item--active:hover .v-icon{
  color: #fff!important;
}



#flat .primary--text{
  color: #FF6B6B!important;
  caret-color: #FF6B6B!important;
}

#flat .v-list-group__header:hover{
  color: #FF6B6B!important;
  caret-color: #FF6B6B!important;
}


#flat .v-list-item__action
{
  margin-right: 15px !important;
}


.button_hover_red_burger:hover {
  /*background-color: #ff575f!important;*/
  /*transition-duration: 0.28s;*/
  /*transition-property: box-shadow, transform, opacity;*/
  /*transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);*/
}
.button_hover_red:hover{
  color:#fff;
  background-color: #FF6B6B!important;
}
.button_hover_red{
  color:#1a535cff;
  background-color: #f7fff7ff;
}


/*.text_hover_red:hover{*/
  /*color: #ff575f!important;*/
/*}*/

.v-list-item__icon:hover{
  color: #FF6B6B!important;
}

/*.v-list-group__header .v-list-item .v-list-item--link:hover{*/
/*  color: #ff575f!important;*/
/*}*/

</style>
