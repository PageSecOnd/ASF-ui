<template>
  <div class="app" :class="[{ 'app--not-authorized': !authenticated, 'app--small-navigation': smallNavigation, 'app--boxed-layout': boxedLayout, 'app--dark-mode': darkMode }, themeClass]">
    <AppHeader></AppHeader>
    <AppNavigation></AppNavigation>
    <AppSideMenu></AppSideMenu>

    <section class="content">
      <router-view></router-view>
      <AppFooter @click="smallNavigation = !smallNavigation"></AppFooter>
    </section>

    <AppModal></AppModal>
    <vue-snotify></vue-snotify>
  </div>
</template>

<script>
  import { mapGetters, mapActions } from 'vuex';
  import AppHeader from './components/App/Header.vue';
  import AppNavigation from './components/App/Navigation.vue';
  import AppFooter from './components/App/Footer.vue';
  import AppSideMenu from './components/App/SideMenu.vue';
  import AppModal from './components/App/Modal.vue';
  import { STATUS } from './utils/getStatus';

  export default {
    name: 'App',
    metaInfo: {
      title: 'ArchiSteamFarm',
      titleTemplate: 'ASF | %s',
    },
    components: {
      AppHeader, AppNavigation, AppFooter, AppSideMenu, AppModal,
    },
    computed: {
      ...mapGetters({
        authenticated: 'auth/authenticated',
        smallNavigation: 'layout/smallNavigation',
        sideMenu: 'layout/sideMenu',
        boxedLayout: 'layout/boxed',
        theme: 'storage/theme',
        darkMode: 'storage/darkMode',
        version: 'asf/version',
        buildVariant: 'asf/buildVariant',
        status: 'auth/status',
      }),
      themeClass() {
        return `theme-${this.theme}`;
      },
    },
    watch: {
      darkMode: {
        immediate: true,
        handler(value) {
          document.documentElement.style.setProperty('--color-background-dark', (value) ? '#0c0c0c' : '#a7a7a7');
        },
      },
      $route: {
        immediate: true,
        handler(value) {
          document.body.style.overflowY = (value.meta.modal) ? 'hidden' : 'auto';
        },
      },
      status: {
        immediate: true,
        handler(value) {
          switch (value) {
            case STATUS.GATEWAY_TIMEOUT:
            case STATUS.RATE_LIMITED:
            case STATUS.UNAUTHORIZED:
            case STATUS.NETWORK_ERROR:
              if (this.$route.name && this.$route.name !== 'setup') this.$router.replace({ name: 'setup' });
              break;
          }
        },
      },
    },
    mounted() {
      window.addEventListener('resize', this.handleResize);
    },
    beforeDestroy() {
      window.removeEventListener('resize', this.handleResize);
    },
    methods: {
      ...mapActions({
        toggleNavigation: 'layout/toggleNavigation',
      }),
      handleResize() {
        const width = document.body.clientWidth;

        if ((width <= 700 && !this.smallNavigation) || (width > 700 && this.smallNavigation)) {
          this.toggleNavigation();
        }
      },
    },
  };
</script>

<style lang="scss">
  @import './style/settings';
  @import './style/components';

  :root {
    --navigation-width: #{$size-navigation};
    --navigation-height: #{$size-navigation-small};
    --color-theme: #{$color-theme-blue};
    --color-theme-dark: #{darken($color-theme-blue, 8)};
    --color-theme-light: #{lighten($color-theme-blue, 8)};
    --color-theme-rgb: 25, 118, 210;
    --color-text: #{$color-text};
    --color-text-dark: #{$color-text-dark};
    --color-text-info: #{$color-text-info};
    --color-text-secondary: #{rgba($color-text-dark, 0.7)};
    --color-text-disabled: #{rgba($color-text-dark, 0.38)};
    --color-border: #{rgba($color-text-dark, 0.12)};
    --color-background: #fafafa;
    --color-background-modal: #fafafa;
    --color-background-light: #fff;
    --color-background-dark: #e0e0e0;
    --color-navigation: #263238;
    --color-navigation-dark: #{darken(#263238, 3)};
    --color-navigation-darker: #{darken(#263238, 8)};

    --color-button-cancel: #{$color-theme-red};
    --color-button-cancel-active: #{darken($color-theme-red, 8)};
    --color-button-default: #757575;

    --color-releases-pre: var(--color-text-secondary);
    --color-releases-code: rgb(27 31 35 / 5%);

    --elevation-1: #{$elevation-1};
    --elevation-2: #{$elevation-2};
    --elevation-3: #{$elevation-3};
    --elevation-4: #{$elevation-4};
    --elevation-6: #{$elevation-6};
    --elevation-8: #{$elevation-8};
  }

  html {
    font-family: 'Roboto', 'Helvetica Neue', Helvetica, Arial, sans-serif;
    font-size: 16px;
    height: 100%;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }

  body {
    background: var(--color-background-dark);
    height: 100%;
    margin: 0;
  }

  @media screen and (max-height: 835px), screen and (max-width: 1366px) {
    html {
      font-size: 14px;
    }
  }

  @media screen and (max-height: 720px), screen and (max-width: 1000px) {
    html {
      font-size: 12px;
    }
  }

  ::-webkit-scrollbar {
    background-color: var(--color-background-dark);
    height: 10px;
    width: 10px;
  }

  ::-webkit-scrollbar-thumb {
    background: #333;
    border-radius: 2px;
  }

  a {
    text-decoration: none;
  }

  .app {
    background: var(--color-background);
    color: var(--color-text-dark);
    width: 100%;
    -webkit-overflow-scrolling: touch;
  }

  .app--small-navigation {
    --navigation-width: #{$size-navigation-small};
  }

  .app--boxed-layout {
    @media screen and (min-width: 1250px) {
      box-shadow: 0 0 25px 0 var(--color-navigation-dark);
      max-width: 1250px;
      margin: 0 auto;
      overflow-x: hidden;
      position: relative;
    }
  }

  .app--dark-mode {
    --color-background: #121212;
    --color-background-light: #1e1e1e;
    --color-background-modal: #2d2d2d;
    --color-text: #e0e0e0;
    --color-text-dark: #ffffff;
    --color-text-info: #ffa726;
    --color-border: rgba(255, 255, 255, .12);
    --color-text-secondary: #{rgba(#ffffff, 0.7)};
    --color-text-disabled: #{rgba(#ffffff, 0.38)};
    --color-button-default: #424242;
    --color-releases-pre: var(--color-text-dark);
    --color-releases-code: rgb(205 217 229 / 15%);
  }

  .theme-blue {
    --color-theme: #{$color-theme-blue};
    --color-theme-dark: #{darken($color-theme-blue, 8)};
    --color-theme-light: #{lighten($color-theme-blue, 8)};
    --color-theme-rgb: 25, 118, 210;
  }

  .theme-red {
    --color-theme: #{$color-theme-red};
    --color-theme-dark: #{darken($color-theme-red, 8)};
    --color-theme-light: #{lighten($color-theme-red, 8)};
    --color-theme-rgb: 211, 47, 47;
    --color-button-cancel: #{darken($color-theme-orange, 8)};
    --color-button-cancel-active: #{$color-theme-orange};
  }

  .theme-teal {
    --color-theme: #{$color-theme-teal};
    --color-theme-dark: #{darken($color-theme-teal, 8)};
    --color-theme-light: #{lighten($color-theme-teal, 8)};
    --color-theme-rgb: 0, 137, 123;
  }

  .theme-purple {
    --color-theme: #{$color-theme-purple};
    --color-theme-dark: #{darken($color-theme-purple, 8)};
    --color-theme-light: #{lighten($color-theme-purple, 8)};
    --color-theme-rgb: 123, 31, 162;
  }

  .theme-green {
    --color-theme: #{$color-theme-green};
    --color-theme-dark: #{darken($color-theme-green, 8)};
    --color-theme-light: #{lighten($color-theme-green, 8)};
    --color-theme-rgb: 56, 142, 60;
  }

  .theme-orange {
    --color-theme: #{$color-theme-orange};
    --color-theme-dark: #{darken($color-theme-orange, 8)};
    --color-theme-light: #{lighten($color-theme-orange, 8)};
    --color-theme-rgb: 245, 124, 0;
  }

  .content {
    box-sizing: border-box;
    display: grid;
    grid-template-rows: 1fr auto;
    grid-template-areas: 'main' 'footer';
    min-height: 100vh;
    padding-left: var(--navigation-width);
    padding-top: var(--navigation-height);
    position: relative;
    transition: ease-in-out padding .3s;

    > main {
      box-sizing: border-box;
      grid-area: main;
      min-width: 0;

      &.main-container--fullheight {
        height: calc(100vh - 2 * var(--navigation-height));
      }
    }

    @media screen and (max-width: 700px) {
      padding-left: $size-navigation-small;
    }
  }
</style>
