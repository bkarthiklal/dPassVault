<template>
  <div>
    <nav>
      <v-toolbar id="nav" text>
        <v-app-bar-nav-icon
          :disabled="userSignedIn === false"
          :color="navColor"
          @click="drawer = !drawer"
        ></v-app-bar-nav-icon>
        <v-toolbar-title class="pl-5">
          THE
          <span class="vault">VAULT</span>
        </v-toolbar-title>
        <v-spacer></v-spacer>
        <v-toolbar-items v-if="pageWidth">
          <v-btn to="/" text>
            <v-icon small class="mr-1">home</v-icon>
            <span class="font-weight-light">Home</span>
          </v-btn>
          <v-btn :disabled="!userSignedIn" to="/Passwords" text>
            <v-icon small class="mr-1">lock</v-icon>
            <span class="font-weight-light">Passwords</span>
          </v-btn>
          <v-btn :disabled="!userSignedIn" to="/Notes" text class="d-none">
            <v-icon small class="mr-1">notes</v-icon>
            <span class="font-weight-light">Notes</span>
          </v-btn>
        </v-toolbar-items>
      </v-toolbar>

      <v-navigation-drawer id="third" v-model="drawer" app dark temporary>
        <v-layout column align-center>
          <v-flex class="mt-5">
            <v-avatar size="120">
              <img class="avatar" :src="avatar" />
            </v-avatar>
            <p class="name white--text subheading mt-3">{{ userName }}</p>
          </v-flex>
          <v-flex v-if="!userSignedIn">
            <v-btn text rounded @click="opened">
              <span>Sign Up</span>
            </v-btn>
          </v-flex>
          <v-flex v-else>
            <v-btn text rounded @click="signOut">
              <span>Sign Out</span>
            </v-btn>
          </v-flex>
        </v-layout>

        <v-list>
          <v-list-item
            v-for="page in pages"
            :key="page.title"
            router
            :to="page.link"
          >
            <v-list-item-action>
              <v-icon>{{ page.icon }}</v-icon>
            </v-list-item-action>

            <v-list-item-content>
              <v-list-item-title>{{ page.title }}</v-list-item-title>
            </v-list-item-content>
          </v-list-item>
        </v-list>

        <v-divider></v-divider>

        <v-layout column align-center>
          <v-btn class="my-4" text rounded @click="dialog = true"
            >Account details</v-btn
          >
        </v-layout>
      </v-navigation-drawer>
    </nav>
    <div id="details">
      <div class="text-xs-center">
        <v-dialog v-model="dialog" width="700">
          <v-card>
            <v-card-title class="headline primary white--text" primary-title
              >Account Details</v-card-title
            >

            <v-list>
              <v-list-item>
                <v-list-item-content>
                  <v-list-item-title>
                    Name:
                    <span class="ml-2">{{ userName }}</span>
                  </v-list-item-title>
                </v-list-item-content>
              </v-list-item>
              <v-list-item>
                <v-list-item-content>
                  <v-list-item-title>
                    Email:
                    <span class="ml-2">{{ email }}</span>
                  </v-list-item-title>
                </v-list-item-content>
              </v-list-item>
              <v-list-item>
                <v-list-item-content>
                  <v-list-item-title>
                    Passwords Saved:
                    <span class="ml-2">{{ totalPasswords }}</span>
                  </v-list-item-title>
                </v-list-item-content>
              </v-list-item>
              <v-list-item>
                <v-list-item-content>
                  <v-list-item-title>
                    Notes Saved:
                    <span class="ml-2">{{ totalNotes }}</span>
                  </v-list-item-title>
                </v-list-item-content>
              </v-list-item>
            </v-list>
            <v-card-actions justify-center>
              <v-spacer></v-spacer>
              <v-btn
                color="red white--text"
                class="mb-3"
                rounded
                :disabled="testAccountLock"
                @click="deleteAccount"
                >Delete Account</v-btn
              >
              <v-spacer></v-spacer>
            </v-card-actions>
            <v-card-text v-if="testAccountLock" class="pl-2"
              >* This is a test account and cannot be deleted</v-card-text
            >
            <v-divider></v-divider>

            <v-card-actions>
              <v-spacer></v-spacer>
              <v-btn color="primary" text @click="dialog = false">Close</v-btn>
            </v-card-actions>
          </v-card>
        </v-dialog>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  components: {},
  props: ['Name'],
  data() {
    return {
      pages: [
        { title: 'Home', icon: 'home', link: '/' },
        { title: 'Passwords', icon: 'lock', link: '/Passwords' },
        // { title: 'Notes', icon: 'note', link: '/Notes' },
      ],
      drawer: false,
      userName: '',
      isLoggedIn: false,
      currentUser: false,
      userSignedIn: false,
      dialog: false,
      email: '',
      totalPasswords: '',
      totalNotes: '',
      id: '',
      avatar: 'https://picsum.photos/200/',
      testAccount: '08dUCWSZ2aX809dc3yB91vwXTDh1',
    }
  },
  computed: {
    colorChange() {
      return this.nav ? '#fff' : '#000'
    },
    pageWidth() {
      return window.innerWidth > 800
    },
    navColor() {
      return window.innerWidth > 800 ? 'black--text' : 'white--text'
    },
    testAccountLock() {
      const uid = ((this.$fire.auth || {}).currentUser || {}).uid || null
      if (uid === this.testAccount) {
        return true
      } else {
        return false
      }
    },
  },
  created() {
    this.$fire.auth.onAuthStateChanged((user) => {
      if (user) {
        this.userSignedIn = true

        this.userName = this.Name
        this.email = user.email
        this.id = user.uid

        if (this.userName === '') {
          this.userName = user.displayName
        }
        this.$fire.firestore
          .collection('users')
          .doc(user.uid)
          .collection('passwords')
          .onSnapshot((res) => {
            this.totalPasswords = res.docs.length
          })

        this.$fire.firestore
          .collection('users')
          .doc(user.uid)
          .collection('notes')
          .onSnapshot((res) => {
            this.totalNotes = res.docs.length
          })
      } else {
        this.userSignedIn = false
        this.userName = ''
      }
    })
  },
  mounted() {},
  methods: {
    opened() {
      if (this.$route.fullPath !== '/') {
        this.$router.push('/')
        setTimeout(() => {
          this.$root.$emit('openUp')
        }, 700)
      } else {
        this.$root.$emit('openUp')
      }

      this.drawer = false
    },
    signOut() {
      this.$fire.auth.signOut().then(() => {
        this.$router.push('/')
        window.location.reload(true)
      })
    },
    deleteAccount() {
      if (confirm('Are You Sure?')) {
        const user = this.$fire.auth.currentUser

        user
          .delete()
          .then(function () {
            alert(user.displayName + ',s account deleted!')
          })
          .catch(function (error) {
            console.log(error)
          })
        this.dialog = false
      }
    },
  },
}
</script>

<style>
.vault {
  font-weight: 300;
}
.nav-color {
  color: #333;
}
.avatar {
  border: #333 3px solid;
}
.name {
  text-align: center;
}
#nav {
  background-color: transparent;
}

@media (max-width: 800px) {
  #nav {
    background-color: #627e8f;
    color: white;
  }
}
</style>
