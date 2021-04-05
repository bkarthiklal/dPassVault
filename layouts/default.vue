<template>
  <v-app>
    <div
      id="background"
      class="background-img"
      :style="{
        backgroundColor: this.color,
        transform: this.skew,
        width: this.width,
        marginLeft: this.margin,
      }"
    ></div>
    <Navbar :name="Name" />

    <v-main class="background-img">
      <transition name="page-animate">
        <!-- <router-view
          @updateUser="updateUser"
          @changePage="updatePage($event)"
        ></router-view> -->
        <nuxt />
      </transition>
    </v-main>
  </v-app>
</template>

<script>
import Navbar from '@/components/Navbar.vue'

export default {
  name: 'App',

  components: {
    Navbar,
  },
  data() {
    return {
      page: 1,
      color: '#fff',
      skew: 'skewX(0)',
      width: '',
      margin: '',
      Name: '',
    }
  },

  created() {},

  methods: {
    updatePage(page) {
      if (window.innerWidth <= 1024) {
        if (page === 1) {
          this.color = 'rgba(255, 255, 255, 0.725)'
          this.skew = 'skewX(30deg)'
          this.width = '0%'
          this.margin = '0'
        }
      } else if (page === 1) {
        this.color = 'rgba(255, 255, 255, 0.725)'
        this.skew = 'skewX(30deg)'
        this.width = '70%'
        this.margin = '-300px'
      }

      if (page === 2) {
        this.color = 'rgba(255, 255, 255, 0.725)'
        this.skew = 'skewX(0)'
        this.width = '100%'
        this.margin = '0'
      }
      if (page === 3) {
        this.color = 'rgba(255, 255, 255, 0.725)'
        this.skew = 'skewX(0)'
        this.width = '100%'
        this.margin = '0'
      }
    },
    updateUser(name) {
      this.Name = name.charAt(0).toUpperCase() + name.slice(1)
    },
  },
}
</script>

<style>
#background {
  width: 0;
  height: 100vh;
  position: absolute;
  background-color: rgba(255, 255, 255, 0.7);
  transform: skewX(0);
  transition: 0.3s ease-in-out;
}
html {
  overflow: hidden;
}
.background-img {
  /* background: url('/bg.jpg'); */
  background: url('https://images.unsplash.com/photo-1508615070457-7baeba4003ab?ixlib=rb-1.2.1&ixid=MXwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHw%3D&auto=format&fit=crop&w=1050&q=80');
  background-size: cover;
}

.page-animate-enter-active {
  animation: coming 0.2s;
  animation-delay: 0.3s;
  opacity: 0;
}

.page-animate-leave-active {
  animation: going 0.3s;
}
.page {
  text-align: center;
  position: fixed;
  width: 100%;
}
.snackbarToFront {
  position: absolute;
  z-index: 10;
}

@keyframes coming {
  from {
    opacity: 0;
    transform: scale(1.1);
  }

  to {
    transform: scale(1);
    opacity: 1;
  }
}

@keyframes going {
  from {
    opacity: 1;
    transform: scale(1);
  }

  to {
    transform: scale(0.9);
    opacity: 0;
  }
}

@media (max-height: 800px) {
  html {
    overflow-y: auto;
  }
}
</style>
