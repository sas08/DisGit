<template>
<v-app>
<v-app-bar
  app
  color="grey darken-3"
  dark
>
  <div class="d-flex align-center mr-2">
    <v-toolbar-title>DisGit</v-toolbar-title>
  </div>
</v-app-bar>

<v-main>
<v-container>
  <v-row class="text-center">
    <v-col cols="12">
      <v-img
        :src="require('../assets/logo.svg')"
        class="my-3"
        contain
        height="200"
      />
    </v-col>

    <v-col class="mb-4">
      <h1 class="display-2 font-weight-bold mb-3">
        Welcome to DisGit
      </h1>

      <p class="subheading font-weight-regular">
        DisGit関係で何かあれば、
        <br>
        <a
          href="https://community.vuetifyjs.com"
          target="_blank"
        >ディスコードサーバー</a>
        へ
      </p>
    </v-col>
    <v-col
      class="mb-5"
      cols="12"
    >
      <h2 style="margin-bottom: 10px;">
        PCでの使用を推奨します
      </h2>
      <v-btn
        color="primary"
        elevation="6"
        large
        outlined
        @click="login"
        >
        <v-icon dark left>
          fab fa-discord
        </v-icon>
        Login
      </v-btn>
    </v-col>
  </v-row>
  <v-row justify="space-around">
    <v-col cols="auto">
      <v-dialog
        transition="dialog-top-transition"
        max-width="600"
        v-model="dialogOAuth"
      >
        <v-card>
          <v-toolbar
            color="primary"
            dark
          >Discord OAuth</v-toolbar>
          <v-card-text>
            <div class="text-h2 pa-12">Please Wait...</div>
          </v-card-text>
          <v-card-actions class="justify-end">
            <v-btn
              text
              @click="dialogOAuth = false"
            >Close</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-col>
  </v-row>
  <v-row justify="space-around">
    <v-col cols="auto">
      <v-dialog
        transition="dialog-top-transition"
        max-width="600"
        v-model="dialogFailed"
      >
        <v-card>
          <v-toolbar
            color="red darken-1"
            dark
          >Discord OAuth</v-toolbar>
          <v-card-text>
            <div class="text-h2 pa-12">OAuth Failed</div>
          </v-card-text>
          <v-card-actions class="justify-end">
            <v-btn
              text
              @click="dialogFailed = false"
            >Close</v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-col>
  </v-row>
</v-container>
</v-main>
</v-app>
</template>

<script>
export default {
  name: 'Home',
  data: () => ({
    dialogOAuth: false,
    dialogFailed: false
  }),
  created: async function () {
    await this.oauth()
  },
  methods: {
    login () {
      window.location.href = `https://discord.com/api/oauth2/authorize?client_id=${process.env.VUE_APP_clientId}&redirect_uri=${process.env.VUE_APP_callback}&response_type=code&scope=identify%20guilds`
    },
    async oauth () {
      if (this.$route.query.code) {
        try {
          this.dialogOAuth = true
          // this.$cookies.remove('userData')
          const code = this.$route.query.code
          const oauthResult = await fetch('https://discord.com/api/oauth2/token', {
            method: 'POST',
            body: new URLSearchParams({
              client_id: process.env.VUE_APP_clientId,
              client_secret: process.env.VUE_APP_clientSecret,
              code,
              grant_type: 'authorization_code',
              redirect_uri: process.env.VUE_APP_redirect,
              scope: 'identify'
            }),
            headers: {
              'Content-Type': 'application/x-www-form-urlencoded'
            }
          })
          const oauthData = await oauthResult.json()
          console.log(oauthData)
          const userResult = await fetch('https://discord.com/api/users/@me', {
            headers: {
              authorization: `${oauthData.token_type} ${oauthData.access_token}`
            }
          })
          const guildResult = await fetch('https://discord.com/api/users/@me/guilds', {
            headers: {
              authorization: `${oauthData.token_type} ${oauthData.access_token}`
            }
          })
          const userData = await userResult.json()
          const guildData = await guildResult.json()

          console.log(userData)
          console.log(guildData)

          this.$store.dispatch('setActionDisData', { user: userData, guild: guildData })
          this.$router.push('/dashboard')
        } catch (error) {
          // NOTE: An unauthorized token will not throw an error;
          // it will return a 401 Unauthorized response in the try block above
          console.error(error)
          this.dialogOAuth = false
          this.dialogFailed = true
        }
      }
    }
  }
}
</script>
