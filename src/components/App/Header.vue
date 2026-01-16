<template>
  <header class="upper-navigation">
    <NavigationBrand></NavigationBrand>

    <div v-tooltip="$t('sidenav-toggle')" class="navigation__button" @click="toggleNavigation">
      <FontAwesomeIcon icon="bars" fixedWidth></FontAwesomeIcon>
    </div>

    <div class="navigation__menu">
      <NavigationLanguageSwitch></NavigationLanguageSwitch>

      <div v-tooltip="$t('sidebar-toggle')" class="navigation__button" @click="toggleSideMenu">
        <FontAwesomeIcon icon="cogs" fixedWidth></FontAwesomeIcon>
      </div>

      <div v-if="status === 'AUTHENTICATED' && password" v-tooltip="$t('logout-title')" class="navigation__button" @click="logout">
        <FontAwesomeIcon icon="sign-out-alt" fixedWidth></FontAwesomeIcon>
      </div>
    </div>
  </header>
</template>

<script>
  import { mapActions, mapGetters } from 'vuex';
  import NavigationBrand from '../Navigation/Brand.vue';
  import NavigationLanguageSwitch from '../Navigation/LanguageSwitch.vue';

  export default {
    name: 'AppHeader',
    components: { NavigationBrand, NavigationLanguageSwitch },
    computed: {
      ...mapGetters({
        status: 'auth/status',
        password: 'auth/password',
        sideMenu: 'layout/sideMenu',
      }),
    },
    watch: {
      sideMenu(value) {
        if (value) window.addEventListener('click', this.onWindowClick);
        else window.removeEventListener('click', this.onWindowClick);
      },
    },
    beforeDestroy() {
      window.removeEventListener('click', this.onWindowClick);
    },
    methods: {
      ...mapActions({
        toggleNavigation: 'layout/toggleNavigation',
        toggleSideMenu: 'layout/toggleSideMenu',
      }),
      async logout() {
        await this.$store.dispatch('auth/setPassword');
        window.location.reload();
      },
      onWindowClick($e) {
        const path = $e.path || $e.composedPath();
        const sideMenu = document.getElementById('side-menu');
        if (path.includes(this.$el) || path.includes(sideMenu)) return;
        this.toggleSideMenu();
      },
    },
  };
</script>

<style lang="scss">
  .upper-navigation {
    background: var(--color-theme);
    display: flex;
    height: var(--navigation-height);
    position: fixed;
    top: 0;
    width: 100%;
    z-index: 1002;
    box-shadow: 0 2px 4px -1px rgba(0, 0, 0, 0.2), 0 4px 5px 0 rgba(0, 0, 0, 0.14), 0 1px 10px 0 rgba(0, 0, 0, 0.12);

    .app--boxed-layout & {
      @media screen and (min-width: 1250px) {
        position: absolute;
      }
    }
  }

  .navigation__button {
    align-items: center;
    color: var(--color-text);
    cursor: pointer;
    display: flex;
    justify-content: center;
    padding: 0 16px;
    transition: background-color 0.2s cubic-bezier(0.4, 0, 0.2, 1);
    position: relative;
    overflow: hidden;

    &::before {
      content: '';
      position: absolute;
      top: 50%;
      left: 50%;
      width: 0;
      height: 0;
      border-radius: 50%;
      background: rgba(255, 255, 255, 0.3);
      transform: translate(-50%, -50%);
      transition: width 0.6s, height 0.6s;
    }

    &:active::before {
      width: 200px;
      height: 200px;
    }

    &.navigation__button--active, &:hover {
      background: rgba(255, 255, 255, 0.1);
    }
  }

  .navigation__menu {
    display: flex;
    justify-content: space-between;
    margin-left: auto;
  }
</style>
