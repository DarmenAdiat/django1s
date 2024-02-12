<template>
  <v-container>
    <v-row class="text-center">
      <v-col class="mb-4">
        <v-col cols='8' style='margin: auto'>
          <v-img
            :src="require('../assets/feedback.png')"
          ></v-img>
        </v-col>
        <v-spacer style=" height: 20px"></v-spacer>
        <p class="subheading font-weight-regular">
          waiting for a nice dashboard...
        </p>
      </v-col>
    </v-row>
  </v-container>
</template>

<script>

export default {
  name: 'MainPage',
  head: {
    title: 'Панель управления',
  },
}
</script>
