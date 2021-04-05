<template>
  <v-app>
    <div id="background" :style="styleObject"></div>
    <Navbar :name="Name" />

    <v-main class="background-img">
      <transition name="page-animate">
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

  computed: {
    styleObject() {
      const homeMobileStyle = [
        'rgba(255, 255, 255, 0.725)',
        'skewX(30deg)',
        '0%',
        '0',
      ]
      const homeWebStyle = [
        'rgba(255, 255, 255, 0.725)',
        'skewX(30deg)',
        '70%',
        '-300px',
      ]
      const passwordtyle = [
        'rgba(255, 255, 255, 0.725)',
        'skewX(0)',
        '100%',
        '0',
      ]
      const createNotesStyle = [...passwordtyle]
      const pageName = this.$route.name
      const generateStyle = (styleArray) => {
        const [color, skew, width, margin] = styleArray
        return {
          backgroundColor: color,
          transform: skew,
          width,
          marginLeft: margin,
        }
      }
      switch (pageName) {
        case 'home':
          if (window.innerWidth <= 1024) {
            return generateStyle(homeMobileStyle)
          }
          return generateStyle(homeWebStyle)
        case 'passwords':
          return generateStyle(passwordtyle)
        case 'notes':
          return generateStyle(createNotesStyle)
        default:
          if (window.innerWidth <= 1024) {
            return generateStyle(homeMobileStyle)
          }
          return generateStyle(homeWebStyle)
      }
    },
  },

  methods: {
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
  background: url('../assets/bg.jpg');
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
