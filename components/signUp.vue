<template>
  <v-layout row justify-center>
    <v-dialog v-model="dialogControl" persistent max-width="600px">
      <v-card>
        <v-card-title>
          <span class="headline">Sign Up</span>
          <v-spacer></v-spacer>
          <v-avatar class="primary" size="70">
            <v-icon color="white" x-large>account_circle</v-icon>
          </v-avatar>
        </v-card-title>
        <v-card-text>
          <v-container grid-list-md>
            <v-layout wrap>
              <v-flex lg12>
                <v-text-field
                  v-model="username"
                  :error-messages="usernameError"
                  label="Username"
                  required
                ></v-text-field>
              </v-flex>

              <v-flex xs12>
                <v-text-field
                  v-model="email"
                  label="Email"
                  :error-messages="emailErrors"
                  required
                ></v-text-field>
              </v-flex>
              <v-flex xs12>
                <v-text-field
                  v-model="password"
                  label="Password*"
                  :type="show ? 'text' : 'password'"
                  :error-messages="passwordErrors"
                  :append-icon="show ? 'visibility' : 'visibility_off'"
                  required
                  @click:append="show = !show"
                ></v-text-field>
              </v-flex>
            </v-layout>
          </v-container>
          <small>*Password must contain uppercase and number</small>
        </v-card-text>
        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="primary" flat @click="closeSignUpDialog">Close</v-btn>
          <v-btn color="primary" :disabled="isDisabled" flat @click="register"
            >Sign Up</v-btn
          >
        </v-card-actions>
      </v-card>
    </v-dialog>
  </v-layout>
</template>

<script>
export default {
  props: ['signUpDialog'],
  data() {
    return {
      show: false,
      username: '',
      email: '',
      password: '',
      accounts: [],
      dialogControl: false,
    }
  },
  computed: {
    usernameError() {
      if (this.username.length > 0 && this.username.length < 3) {
        return '3 Characters Minimum'
      }
      return ''
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
      const uppercase = /[A-Z]/.test(this.password)
      const numbers = /[0-9]/.test(this.password)
      const space = /[ ]/.test(this.password)
      if (this.password.length > 0) {
        if (space) {
          return 'No Spaces Allowed'
        }

        if (this.password.length < 8) {
          return 'To Weak'
        }

        if (!uppercase || !numbers) {
          return 'To Weak'
        }
      }
      return ''
    },
    isDisabled() {
      if (
        this.password.length === 0 ||
        this.email.length < 3 ||
        this.emailErrors === 'Email not valid' ||
        this.passwordErrors === 'To Weak' ||
        this.passwordErrors === 'No Spaces Allowed' ||
        this.username.length < 3
      ) {
        return true
      } else {
        return false
      }
    },
  },
  watch: {
    dialogControl(value) {
      this.$parent.signUpDialog = value
    },
    signUpDialog(value) {
      this.dialogControl = value
    },
  },
  methods: {
    closeSignUpDialog() {
      this.$emit('closeSignUpDialog')
    },

    register(e) {
      e.preventDefault()

      this.$fire.auth
        .createUserWithEmailAndPassword(this.email, this.password)
        .then((user) => {
          user.user
            .updateProfile({
              displayName:
                this.username.charAt(0).toUpperCase() + this.username.slice(1),
            })
            .then(() => {
              const user = this.$fire.auth.currentUser
              this.$fire.firestore.collection('users').doc(user.uid).set({
                id: user.uid,
                name: user.displayName,
                email: user.email,
              })
            })
        })

      this.$emit('updateUser', this.username)
      this.$emit('closeSignUpDialog')
    },
  },
}
</script>
