<template>
  <div id="passwordPage" class="page">
    <passwordCreator
      ref="edit"
      :change-password="editOpen"
      :password-dialog="passwordDialog"
      @dialogToggle="closeDialog()"
      @passwordDialogUpdated="dialogControl"
    />
    <h1 v-if="pageWidth" class="subheading grey--text">VAULT</h1>

    <v-container class="pass-wrap">
      <v-layout wrap>
        <v-flex class="add-password">
          <v-btn
            class="font-weight-light"
            color="grey"
            text
            rounded
            @click="passwordDialog = true"
          >
            <span>add password</span>
            <v-icon small>add</v-icon>
          </v-btn>
        </v-flex>

        <v-flex>
          <v-toolbar dense text class="tools password-counter">
            <v-toolbar-title class="tb-text subheading"
              >favorites</v-toolbar-title
            >
            <v-btn text rounded @click="filterFavorites()">
              <v-icon color="pink">favorite</v-icon>
              <span class="pink--text">{{ favoriteCount }}</span>
            </v-btn>

            <v-divider v-if="pageWidth" class="ml-3" inset vertical></v-divider>
            <v-spacer v-if="pageWidth"></v-spacer>
            <v-toolbar-title v-if="pageWidth" class="tb-text subheading"
              >Passwords</v-toolbar-title
            >

            <div v-if="pageWidth">
              <v-avatar size="35" class="green white--text ml-4 ma-2">{{
                this.strong
              }}</v-avatar>
              <v-avatar size="35" class="orange white--text ma-2">{{
                this.medium
              }}</v-avatar>
              <v-avatar size="35" class="red white--text ma-2">{{
                this.weak
              }}</v-avatar>
            </div>
            <v-spacer v-if="pageWidth"></v-spacer>
            <v-divider v-if="pageWidth" class="ml-3" inset vertical></v-divider>
            <v-spacer v-if="!pageWidth"></v-spacer>
            <v-toolbar-title class="subheading">Showing</v-toolbar-title>

            <v-speed-dial
              v-model="fab"
              :direction="direction"
              :open-on-hover="hover"
              :transition="transition"
            >
              <template #activator>
                <v-btn v-model="fab" :color="fabColor" dark fab>
                  <v-icon v-if="isIcon" :color="iconColor">{{ icon }}</v-icon>
                  <span v-else>{{ strength }}</span>
                </v-btn>
              </template>
              <v-btn fab small color="white" @click="filterButtons(1)">
                <v-icon color="pink">favorite</v-icon>
              </v-btn>
              <v-btn fab dark small color="green" @click="filterButtons(2)">
                <span>{{ this.strong }}</span>
              </v-btn>
              <v-btn fab dark small color="orange" @click="filterButtons(3)">
                <span>{{ this.medium }}</span>
              </v-btn>
              <v-btn fab dark small color="red" @click="filterButtons(4)">
                <span>{{ this.weak }}</span>
              </v-btn>
              <v-btn fab small color="secondary" @click="filterButtons(5)">
                <span>All</span>
              </v-btn>
            </v-speed-dial>
          </v-toolbar>
        </v-flex>
      </v-layout>

      <v-divider class="my-3"></v-divider>

      <v-layout id="cards" row wrap>
        <template v-for="password in passwords">
          <v-flex
            v-if="password.show"
            :key="password.id"
            xs12
            sm6
            md4
            lg3
            class="card"
          >
            <v-hover>
              <v-card
                slot-scope="{ hover }"
                :class="`elevation-${hover ? 12 : 2}`"
                class="card text-xs-center ma-3"
              >
                <v-flex xs12 sm3>
                  <v-btn
                    text
                    icon
                    :color="password.favorite"
                    @click="isFavorite(password.id, password.favorite)"
                  >
                    <v-icon>favorite</v-icon>
                  </v-btn>
                </v-flex>
                <v-responsive class="pt-4">
                  <v-avatar :class="`${password.strength}`" size="100">
                    <img src="~/assets/lock-default.png" />
                  </v-avatar>
                </v-responsive>
                <v-card-text>
                  <div class="subheading">{{ password.website }}</div>
                  <v-divider class="my-3"></v-divider>
                  <v-chip :class="`${password.strength}`" text-color="white">{{
                    password.strength
                  }}</v-chip>
                </v-card-text>
                <v-card-actions>
                  <v-btn
                    text
                    color="grey"
                    @click="
                      edit(password.id, password.website, password.password)
                    "
                  >
                    <v-icon small left>create</v-icon>
                    <span>Edit</span>
                  </v-btn>
                  <v-spacer></v-spacer>
                  <v-btn
                    text
                    color="grey"
                    @click="getPassword(password.website, password.password)"
                  >
                    <v-icon small left>lock_open</v-icon>
                    <span>Show</span>
                  </v-btn>
                </v-card-actions>
              </v-card>
            </v-hover>
          </v-flex>
        </template>
      </v-layout>
      <v-layout>
        <v-dialog v-model="dialog" max-width="290">
          <v-card>
            <v-card-title class="headline primary white--text">{{
              this.name
            }}</v-card-title>
            <p class="show subheading title black--text text-xs-center mt-3">
              {{ text }}
            </p>

            <v-divider></v-divider>

            <v-card-text
              id="copy-pass"
              ref="input"
              class="pass-text text-xs-center"
              color="grey--text"
              >{{ this.pass }}</v-card-text
            >

            <v-card-actions>
              <v-spacer></v-spacer>

              <v-btn color="primary" text="text" @click="doCopy">
                <span>Copy</span>
              </v-btn>

              <v-btn color="primary" text="text" @click="dialog = false">
                <span>Close</span>
              </v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </v-layout>
    </v-container>
  </div>
</template>

<script>
import { setTimeout } from 'timers'
import PasswordCreator from '@/components/passwordCreate.vue'
export default {
  components: {
    PasswordCreator,
  },

  data() {
    return {
      direction: 'bottom',
      fab: false,
      fling: false,
      hover: true,
      tabs: null,
      transition: 'slide-y-transition',
      passwords: [],
      tempPasswords: [],
      page: 2,
      color: '',
      dialog: false,
      onlyFavorite: false,
      name: '',
      pass: '',
      passwordDialog: false,
      text: 'Password',
      editOpen: false,
      updateID: '',
      fabShowing: 0,
      icon: '',
      fabColor: '#627E8F',
      isIcon: false,
      iconColor: '#fff',
      strength: 'All',
      jeff: true,
    }
  },
  computed: {
    strong() {
      return this.passwords.filter((password) => password.strength === 'strong')
        .length
    },
    medium() {
      return this.passwords.filter((password) => password.strength === 'medium')
        .length
    },
    weak() {
      return this.passwords.filter((password) => password.strength === 'weak')
        .length
    },
    favoriteCount() {
      return this.passwords.filter((password) => password.favorite === 'pink')
        .length
    },
    pageWidth() {
      return window.innerWidth > 800
    },
  },

  beforeCreate() {},
  mounted() {},
  updated() {},
  created() {
    this.$emit('changePage', 2)
    const user = this.$fire.auth.currentUser
    this.$fire.firestore
      .collection('users')
      .doc(user.uid)
      .collection('passwords')
      .onSnapshot((res) => {
        const changes = res.docChanges()

        changes.forEach((change) => {
          if (change.type === 'added') {
            this.passwords.push({
              ...change.doc.data(),
              id: change.doc.id,
            })
          } else if (change.type === 'modified') {
            this.passwords.forEach((password) => {
              if (password.id === change.doc.id) {
                password.website = change.doc.data().website
                password.password = change.doc.data().password
                password.strength = change.doc.data().strength
                password.favorite = change.doc.data().favorite
              }
            })
          } else if (change.type === 'removed') {
            const removedPassword = this.passwords.filter(
              (password) => password.id !== change.doc.id
            )

            this.passwords = removedPassword
          }
        })
      })
  },
  methods: {
    getPassword(name, password) {
      this.dialog = true
      this.name = name
      this.pass = password
    },
    closeDialog() {
      this.passwordDialog = false
      this.editOpen = false
    },
    doCopy() {
      const input = this.$refs.input

      this.$copyText(this.pass, input)
      this.text = 'Copied!'

      setTimeout(() => {
        this.text = 'Password'
      }, 2000)
    },
    edit(id, website, password) {
      this.passwordDialog = true
      this.editOpen = true
      this.$refs.edit.editPassword(id, website, password)
      this.updateID = id
    },
    isFavorite(favoriteID, favoriteColor) {
      this.$refs.edit.changeFavorite(favoriteID, favoriteColor)
    },
    filterButtons(num) {
      const user = this.$fire.auth.currentUser
      this.$fire.firestore
        .collection('users')
        .doc(user.uid)
        .collection('passwords')
        .onSnapshot((data) => {
          if (num === 1) {
            this.isIcon = true
            this.passwords.forEach((password) => {
              password.show = true
              if (password.favorite !== 'pink') {
                password.show = false
              }
            })
            this.icon = 'favorite'
            this.iconColor = 'pink'
            this.fabColor = '#fff'
          }
          if (num === 2) {
            this.isIcon = false
            this.passwords.forEach((password) => {
              password.show = true
              if (password.strength !== 'strong') {
                password.show = false
              }
            })
            this.strength = this.strong
            this.fabColor = 'green'
          }
          if (num === 3) {
            this.isIcon = false
            this.passwords.forEach((password) => {
              password.show = true
              if (password.strength !== 'medium') {
                password.show = false
              }
            })
            this.strength = this.medium
            this.fabColor = 'orange'
          }
          if (num === 4) {
            this.isIcon = false
            this.passwords.forEach((password) => {
              password.show = true
              if (password.strength !== 'weak') {
                password.show = false
              }
            })
            this.strength = this.weak
            this.fabColor = 'red'
          }
          if (num === 5) {
            this.isIcon = false
            this.passwords.forEach((password) => {
              password.show = true
            })
            this.strength = 'All'
            this.fabColor = '#627E8F'
          }
        })
    },
    dialogControl(value) {
      this.passwordDialog = value
    },
    filterFavorites() {},
  },
}
</script>

<style scoped>
.pass-text {
  letter-spacing: 3px;
  font-weight: 700;
}
.password-counter {
  border-radius: 50px;
  background-color: rgb(255, 255, 255);
}
#cards {
  background-color: rgba(233, 233, 233, 0.363);
  margin-bottom: 50px;
}
.tools {
  position: relative;
  z-index: 2;
}

#passwordPage {
  overflow-y: auto;
  height: 100vh;
}

.strong {
  background-color: green;
}
.medium {
  background-color: orange;
}
.weak {
  background-color: red;
}
.pass-wrap {
  margin-top: 50px;
}

@media (max-width: 800px) {
  .add-password {
    margin: 30px 0;
  }
  .pass-wrap {
    margin: 0;
  }
  #cards {
    margin-bottom: 100px;
  }
}

@media (max-width: 600px) {
  .tools {
    display: flex;
  }
}
</style>
