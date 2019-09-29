<template>
  <v-layout>
    <v-flex class="text-center">
      <v-row align="center">
        <v-col class="text-center" cols="12" sm="4">
          <v-img
            src="img/infograph2.png"
            aspect-ratio="1"
          ></v-img>
          <v-btn large>Send</v-btn>
        </v-col>
      </v-row>
    </v-flex>
  </v-layout>
</template>

<script>
  export default {
    head() {
      return {
        link: [
          {
            rel: 'stylesheet',
            href: 'https://stackpath.bootstrapcdn.com/font-awesome/4.7.0/css/font-awesome.min.css'
          }
        ]
      }
    }
  }
</script>

<style scoped>
  h2 {
    font-family: Roboto, sans-serif;
    font-size: 18px;
  }
</style>





