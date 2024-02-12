<template>
  <v-menu v-model="showMenu" offset-y>
    <template #activator="{ on }">
      <v-btn class="btnLocaleActivation primary--text" text v-on="on" >
        <v-icon>mdi-earth</v-icon>
        &nbsp;{{ displayLocale }}
      </v-btn>
    </template>
    <v-list>
      <v-list-item
          v-for="(item, i) in langs"
          :key="`Lang${i}`"
          :value="item.lang"
          :class="[item.class]"
          active-class="primary"
          @click="switchLocale(item.lang)"
      >
        <country-flag :country="item.flag" size="small" />
        <v-list-item-title class="ml-3">{{
            item.lang.toUpperCase()
          }}</v-list-item-title>
      </v-list-item>
    </v-list>
  </v-menu>
</template>

<script>
import CountryFlag from 'vue-country-flag'

export default {
  components: {
    CountryFlag
  },

  data() {
    return {
      showMenu: false,
      langs: [
        {
          lang: 'ru',
          flag: 'ru',
          class: 'btnRU'
        },
        {
          lang: 'en',
          flag: 'gb',
          class: 'btnEN'
        },
        {
          lang: 'kz',
          flag: 'kz',
          class: 'btnKZ'
        },
      ]
    }
  },
  computed: {
    availableLocales () {
      return this.$i18n.locales.filter(i => i.code !== this.$i18n.locale)
    },
    displayLocale() {
      return this.$i18n.locale
    }
  },
  methods: {
    switchLocale(lang) {
      this.$i18n.setLocale(lang)
      // this.$router.push({ path: this.switchLocalePath() });
      // alert(this.switchLocalePath())
      this.reloadOnSwitch(this.switchLocalePath())
    },
    reloadOnSwitch(pathTo){
      // this.$router.push(pathTo);
      window.location.replace(pathTo)
    }
  },

}
</script>

<style>
.flag {
  width: 118px !important;
}
</style>