<template>
  <v-app>
    <v-app-bar
      app
      class="bg-app-bar"
      dark
    >
      <div class="np_fs22 font-weight-medium">
       Nipa.Cloud x Siwakorn.J
      </div>
    </v-app-bar>

    <v-main>
      <router-view/>
    </v-main>
  </v-app>
</template>

<script>

export default {
  name: 'App',

  data: () => ({
    //
  }),
};
</script>
<style>
@import url('https://fonts.googleapis.com/css2?family=Prompt:wght@400;500&display=swap');
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
  font-family: 'Prompt', sans-serif;
}
.v-application--wrap {
  background: #F1F2F5;  
}
.np_fs14 {
  font-size: 14px !important;
}
.np_fs18 {
  font-size: 18px !important;
}
.np_fs22 {
  font-size: 22px !important;
}
.bg-app-bar {
  background: linear-gradient(150deg, rgba(224,26,112,1) 0%, rgba(85,43,81,1) 100%);
}
</style>
