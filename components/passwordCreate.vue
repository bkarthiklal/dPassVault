<template>
  <div class="create-password">
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
    <v-layout row justify-center>
      <v-dialog v-model="passwordDialog" persistent max-width="600px">
        <v-card>
          <v-card-title>
            <span class="headline">Add Password</span>
            <v-spacer></v-spacer>

            <v-btn
              v-if="changePassword"
              outline
              fab
              color="primary"
              @click="deletePassword()"
            >
              <v-icon>delete</v-icon>
            </v-btn>
          </v-card-title>
          <v-card-text>
            <v-container grid-list-md>
              <v-layout wrap>
                <v-flex xs12 sm6 md4>
                  <v-text-field
                    v-model="website"
                    label="Website Name"
                    required
                  ></v-text-field>
                </v-flex>
                <v-spacer></v-spacer>
                <div>
                  <v-progress-circular
                    :rotate="360"
                    :size="100"
                    :width="15"
                    :value="scorePassword"
                    :color="color"
                    >{{ strength }}</v-progress-circular
                  >
                </div>

                <v-flex xs12>
                  <v-text-field
                    v-model="password"
                    label="Password"
                    :type="show ? 'text' : 'password'"
                    :error-messages="noSpaces"
                    :append-icon="show ? 'visibility' : 'visibility_off'"
                    required
                    @click:append="show = !show"
                  ></v-text-field>
                </v-flex>
                <v-flex xs12>
                  <v-btn class="primary" flat round @click="generate"
                    >Generate</v-btn
                  >
                </v-flex>
              </v-layout>
            </v-container>
          </v-card-text>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn color="primary" flat @click="dialogToggle">Close</v-btn>
            <v-btn
              color="primary"
              flat
              :disabled="isDisabled"
              @click="addPassword"
              >Add</v-btn
            >
          </v-card-actions>
        </v-card>
      </v-dialog>
    </v-layout>
  </div>
</template>

<script>
import firebase from 'firebase'
import db from './firebaseInit'
export default {
  components: {},
  props: ['passwordDialog', 'changePassword'],
  data() {
    return {
      website: '',
      password: '',
      show: false,
      value: 0,
      id: '',
      spaces: false,
      userId: '',
      userSignedIn: false,
      snackbar: false,
      snackbarMessage: '',
      timeout: 3000,
    }
  },

  computed: {
    pageWidth() {
      return window.innerWidth > 800
    },
    scorePassword() {
      let score = 0
      if (this.password === '') return score

      const letters = {}

      for (let i = 0; i < this.password.length; i++) {
        letters[this.password[i]] = (letters[this.password[i]] || 0) + 1
        score += 5.0 / letters[this.password[i]]
      }
      const variations = {
        digits: /\d/.test(this.password),
        lower: /[a-z]/.test(this.password),
        upper: /[A-Z]/.test(this.password),
        special: /\W/.test(this.password),
      }

      let variationsCount = 0
      for (const check in variations) {
        variationsCount += variations[check] === true ? 1 : 0
      }
      score += (variationsCount - 1) * 10

      return parseInt(score)
    },

    strength() {
      if (this.scorePassword < 50) {
        return 'Weak'
      } else if (this.scorePassword >= 50 && this.scorePassword < 100) {
        return 'Medium'
      } else {
        return 'Strong'
      }
    },
    color() {
      if (this.strength === 'Weak') {
        return 'red'
      } else if (this.strength === 'Medium') {
        return 'orange'
      } else {
        return 'green'
      }
    },
    isDisabled() {
      if (this.spaces) {
        return true
      }
      if (this.password.length > 3 && this.website.length > 2) {
        return false
      } else {
        return true
      }
    },
    noSpaces() {
      const reg = new RegExp(/ /)
      this.spaces = reg.test(this.password)
      if (this.spaces) {
        return 'No spaces allowed!'
      }
    },
  },
  watch: {},

  created() {
    firebase.auth().onAuthStateChanged((user) => {
      if (user) {
        this.userId = user.uid
        this.userSignedIn = true
      } else {
        this.userSignedIn = false
      }
    })
  },
  mounted() {},
  methods: {
    dialogToggle() {
      this.$emit('dialogToggle')
      this.password = ''
      this.website = ''
    },
    generate() {
      let password = ''
      let character

      for (let i = 0; i < 20; i++) {
        !password.includes(
          (character = String.fromCharCode(
            Math.floor(Math.random() * 94) + 33
          )),
          Math.floor(password.length / 94) * 94
        )
        password += character
      }
      this.password = password
    },

    addPassword() {
      if (this.userSignedIn) {
        if (this.changePassword) {
          db.collection('users')
            .doc(this.userId)
            .collection('passwords')
            .doc(this.id)
            .update({
              website: this.website,
              password: this.password,
              strength: this.strength.toLowerCase(),
            })
            .then(() => {
              this.password = ''
              this.website = ''
              this.snackbar = true
              this.snackbarMessage = 'Password Updated!'
            })
        } else {
          db.collection('users')
            .doc(this.userId)
            .collection('passwords')
            .add({
              website: this.website,
              password: this.password,
              strength: this.strength.toLowerCase(),
              favorite: 'grey',
              show: true,
            })
            .then(() => {
              this.password = ''
              this.website = ''
              this.snackbar = true
              this.snackbarMessage = 'Password Saved!'
            })
        }
      }
    },
    editPassword(id, website, password) {
      this.id = id
      this.website = website
      this.password = password
    },
    deletePassword() {
      if (this.userSignedIn) {
        if (confirm('Are You Sure?')) {
          db.collection('users')
            .doc(this.userId)
            .collection('passwords')
            .doc(this.id)
            .delete()
            .then(() => {
              this.snackbar = true
              this.snackbarMessage = 'Password Deleted!'
              this.dialogToggle()
            })
        }
      }
    },

    changeFavorite(id, favoriteColor) {
      if (favoriteColor === 'pink') {
        db.collection('users')
          .doc(this.userId)
          .collection('passwords')
          .doc(id)
          .update({ favorite: 'grey' })
      } else {
        db.collection('users')
          .doc(this.userId)
          .collection('passwords')
          .doc(id)
          .update({ favorite: 'pink' })
      }
    },
  },
}
</script>

<style></style>
