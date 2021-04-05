<template>
  <div class="home page">
    <v-snackbar
      v-if="pageWidth"
      v-model="snackbar"
      class="snackbarToFront"
      :timeout="timeout"
      top
    >
      {{ snackbarMessage }}
      <v-btn color="primary" flat @click="snackbar = false">Close</v-btn>
    </v-snackbar>
    <v-snackbar v-else v-model="snackbar" :timeout="timeout" bottom>
      {{ snackbarMessage }}
      <v-btn color="primary" flat @click="snackbar = false">Close</v-btn>
    </v-snackbar>
    <signUp
      ref="signUpBlock"
      :sign-up-dialog="signUpDialog"
      @updateUser="updateUser"
      @closeSignUpDialog="closeSignUpDialog()"
    />
    <v-container class="homeContainer">
      <v-layout wrap justify-space-around row fill-height>
        <v-flex v-if="!userSignedIn" class="sign-in-wrap" xs12 sm12 md5 lg5>
          <v-icon color="secondary" class="lock">lock</v-icon>
          <form>
            <v-text-field
              v-model="email"
              :error-messages="emailErrors"
              label="E-mail"
              required
            ></v-text-field>
            <v-text-field
              v-model="password"
              :error-messages="passwordErrors"
              label="Password"
              required
              :type="show ? 'text' : 'password'"
              :append-icon="show ? 'visibility' : 'visibility_off'"
              @click:append="show = !show"
            ></v-text-field>
          </form>
          <v-btn flat round class="my-3 secondary" large @click="signIn"
            >Login</v-btn
          >
          <h3>
            Don't have an account?
            <span class="sign-up" @click="openSignUp">sign up</span>
          </h3>
        </v-flex>

        <v-flex v-else class="box signed-in-wrap" xs12 sm12 md5 lg5>
          <v-avatar class="avatar-wrap" size="150">
            <img class="avatar" src="/default.png" />
          </v-avatar>

          <h1 class="my-4">Welcome Back {{ userName }}</h1>
          <v-btn flat round class="my-3 secondary" large @click="signOut"
            >Sign Out</v-btn
          >
        </v-flex>

        <v-flex xs12 sm12 md5 lg5 class="welcome-wrap">
          <h3 class="wel-text mb-3 white--text">Welcome To The Vault</h3>

          <span class="subheading font-weight-thin white--text">
            Create and store strong passwords and never have to worry about
            forgetting them, you will get a strength check on your password or
            you can use the password generator and automatically get a strong
            password.
          </span>

          <v-divider
            style="width: 500px; margin: auto"
            class="my-5"
          ></v-divider>

          <div class="white--text title mb-3">Create password now!</div>

          <v-btn
            :disabled="!userSignedIn"
            flat
            round
            class="my-3 primary"
            large
            @click="createPassword"
            >create</v-btn
          >
        </v-flex>
      </v-layout>
    </v-container>
  </div>
</template>

<script>
import signUp from '@/components/signUp.vue'
export default {
  components: {
    signUp,
  },
  data() {
    return {
      email: '',
      password: '',
      signUpDialog: false,
      show: false,
      userSignedIn: false,
      userName: '',
      avatar: `https://api.adorable.io/avatars/167/`,
      id: '',
      snackbar: false,
      snackbarMessage: '',
      timeout: 5000,
    }
  },

  computed: {
    pageWidth() {
      return window.innerWidth > 1024
    },
    emailErrors() {
      // eslint-disable-next-line prefer-regex-literals
      const regex = new RegExp(/@/)
      // eslint-disable-next-line prefer-regex-literals
      const dotCom = new RegExp(/[.]/)
      if (this.email.length > 0) {
        if (
          regex.test(this.email) === false ||
          dotCom.test(this.email) === false
        ) {
          return 'Email not valid'
        }
      }
      return ''
    },
    passwordErrors() {
      if (this.password.length > 0 && this.password.length < 6) {
        return 'To short'
      }
      return ''
    },
  },
  created() {
    this.$emit('changePage', 1)
    this.$fire.auth.onAuthStateChanged((user) => {
      if (user) {
        this.userSignedIn = true

        if (this.userName === '') {
          this.userName = user.displayName
        }
        this.id = user.uid
      } else {
        this.userSignedIn = false
      }
    })
  },
  mounted() {
    this.$root.$on('openUp', () => {
      this.signUpDialog = true
      this.$refs.signUpBlock.dialogControl = this.signUpDialog
    })
  },

  methods: {
    openSignUp() {
      this.signUpDialog = true
      this.$refs.signUpBlock.dialogControl = this.signUpDialog
    },
    closeSignUpDialog() {
      this.signUpDialog = false
      this.$refs.signUpBlock.dialogControl = this.signUpDialog
    },
    signIn() {
      this.$fire.auth
        .signInWithEmailAndPassword(this.email, this.password)
        .catch((error) => {
          const errorCode = error.code
          // const errorMessage = error.message
          this.snackbar = true
          this.snackbarMessage = 'user not found! ' + errorCode
        })
    },
    signOut() {
      this.$fire.auth.signOut().then(() => {
        this.userSignedIn = false
        window.location.reload(true)
      })
    },
    updateUser(name) {
      this.userName = name.charAt(0).toUpperCase() + name.slice(1)
      this.$emit('updateUser', name)
    },
    createPassword() {
      this.$router.push('/Passwords')
    },
  },
}
</script>

<style scoped>
.homeContainer {
  min-height: 90vh;
  display: flex;
  align-items: center;
}
.sign-up {
  cursor: pointer;
  font-weight: 300;
  margin-left: 10px;
}
.wel-text {
  font-size: 44px;
}

.lock {
  font-size: 80px;
  width: 120px;
  height: 120px;
  border: solid 3px;
  border-radius: 50%;
}
.avatar {
  border: black 1px solid;
}
@media (max-height: 800px) {
  .sign-in-wrap,
  .signed-in-wrap,
  .welcome-wrap {
    margin: 50px 0;
  }
  .page {
    position: static;
  }
}
@media (max-width: 600px) {
  .wel-text {
    font-size: 26px;
  }
}
</style>
