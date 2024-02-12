<template>
  <div class="about">
    <v-col cols='10' style='margin: auto'>
      <v-img
        :src="require('../assets/team.jpg')"
      ></v-img>
    </v-col>
  </div>
</template>

<script>
export default {
  name: "Team",
  head: {
    title: 'Команда',
  },
}
</script>

<style scoped>

</style>
