<template>
  <div>
    <nav>
      <div class="win32-titlebar">
        <div class="title">YesPlayMusic</div>
        <div class="controls">
          <div
            class="button minimize codicon codicon-chrome-minimize"
            @click="windowMinimize"
          ></div>
          <div
            class="button max-restore codicon"
            @click="windowMaxRestore"
            :class="{
              'codicon-chrome-restore': !isWindowMaximized,
              'codicon-chrome-maximize': isWindowMaximized,
            }"
          ></div>
          <div
            class="button close codicon codicon-chrome-close"
            @click="windowClose"
          ></div>
        </div>
      </div>
      <div class="navigation-buttons">
        <button-icon @click.native="go('back')"
          ><svg-icon icon-class="arrow-left"
        /></button-icon>
        <button-icon @click.native="go('forward')"
          ><svg-icon icon-class="arrow-right"
        /></button-icon>
      </div>
      <div class="navigation-links">
        <router-link to="/" :class="{ active: this.$route.name === 'home' }">{{
          $t("nav.home")
        }}</router-link>
        <router-link
          to="/explore"
          :class="{ active: this.$route.name === 'explore' }"
          >{{ $t("nav.explore") }}</router-link
        >
        <router-link
          to="/library"
          :class="{ active: this.$route.name === 'library' }"
          >{{ $t("nav.library") }}</router-link
        >
      </div>
      <div class="right-part">
        <div class="search-box">
          <div class="container" :class="{ active: inputFocus }">
            <svg-icon icon-class="search" />
            <div class="input">
              <input
                ref="searchInput"
                :placeholder="inputFocus ? '' : $t('nav.search')"
                v-model="keywords"
                @keydown.enter="doSearch"
                @focus="inputFocus = true"
                @blur="inputFocus = false"
              />
            </div>
          </div>
        </div>
        <img class="avatar" :src="avatarUrl" @click="showUserProfileMenu" />
      </div>
    </nav>

    <ContextMenu ref="userProfileMenu">
      <div class="item" @click="toSettings">
        <svg-icon icon-class="settings" />
        {{ $t("library.userProfileMenu.settings") }}
      </div>
      <div class="item" @click="toLogin" v-if="!isLooseLoggedIn">
        <svg-icon icon-class="login" />
        {{ $t("login.login") }}
      </div>
      <div class="item" @click="logout" v-if="isLooseLoggedIn">
        <svg-icon icon-class="logout" />
        {{ $t("library.userProfileMenu.logout") }}
      </div>
      <hr />
      <div class="item" @click="toGitHub">
        <svg-icon icon-class="github" />
        {{ $t("nav.github") }}
      </div>
    </ContextMenu>
  </div>
</template>

<script>
import { mapState } from "vuex";
import { isLooseLoggedIn, doLogout } from "@/utils/auth";

// import icons for win32 title bar
// icons by https://github.com/microsoft/vscode-codicons
import "vscode-codicons/dist/codicon.css";

import ContextMenu from "@/components/ContextMenu.vue";
import ButtonIcon from "@/components/ButtonIcon.vue";

const electron =
  process.env.IS_ELECTRON === true ? window.require("electron") : null;
const ipcRenderer =
  process.env.IS_ELECTRON === true ? electron.ipcRenderer : null;

export default {
  name: "Navbar",
  components: {
    ButtonIcon,
    ContextMenu,
  },
  data() {
    return {
      inputFocus: false,
      langs: ["zh-CN", "en", "tr"],
      keywords: "",
      isWindowMaximized: false,
    };
  },
  computed: {
    ...mapState(["settings", "data"]),
    isLooseLoggedIn() {
      return isLooseLoggedIn();
    },
    avatarUrl() {
      return this.data?.user?.avatarUrl && this.isLooseLoggedIn
        ? `${this.data?.user?.avatarUrl}?param=512y512`
        : "http://s4.music.126.net/style/web2/img/default/default_avatar.jpg?param=60y60";
    },
  },
  created() {
    if (process.env.IS_ELECTRON === true) {
      ipcRenderer.on("isMaximized", (event, value) => {
        this.isWindowMaximized = value;
      });
    }
  },
  methods: {
    go(where) {
      if (where === "back") this.$router.go(-1);
      else this.$router.go(1);
    },
    doSearch() {
      if (!this.keywords) return;
      if (
        this.$route.name === "search" &&
        this.$route.params.keywords === this.keywords
      ) {
        return;
      }
      this.$router.push({
        name: "search",
        params: { keywords: this.keywords },
      });
    },
    showUserProfileMenu(e) {
      this.$refs.userProfileMenu.openMenu(e);
    },
    logout() {
      if (!confirm("确定要退出登录吗？")) return;
      doLogout();
      this.$router.push({ name: "home" });
    },
    toSettings() {
      this.$router.push({ name: "settings" });
    },
    toGitHub() {
      window.open("https://github.com/qier222/YesPlayMusic");
    },
    toLogin() {
      this.$router.push({ name: "login" });
    },
    windowMinimize() {
      ipcRenderer.send("minimize");
    },
    windowMaxRestore() {
      ipcRenderer.send("maximizeOrUnmaximize");
    },
    windowClose() {
      ipcRenderer.send("close");
    },
  },
};
</script>

<style lang="scss" scoped>
nav {
  position: fixed;
  top: 0;
  right: 0;
  left: 0;
  display: flex;
  justify-content: space-between;
  align-items: center;
  height: 64px;
  padding: {
    right: 10vw;
    left: 10vw;
  }
  backdrop-filter: saturate(180%) blur(20px);

  background-color: var(--color-navbar-bg);
  z-index: 100;
  -webkit-app-region: drag;
}

@media (max-width: 1336px) {
  nav {
    padding: 0 5vw;
  }
}

@supports (-moz-appearance: none) {
  nav {
    background-color: var(--color-body-bg);
  }
}

.win32-titlebar {
  display: none;
}

[data-electron-os="win32"] {
  nav {
    padding-top: 20px;
    -webkit-app-region: no-drag;
  }
  .win32-titlebar {
    color: var(--color-text);
    position: fixed;
    left: 0;
    top: 0;
    right: 0;
    -webkit-app-region: drag;
    display: flex;
    align-items: center;
    --hover: #e6e6e6;
    --active: #cccccc;

    .title {
      padding: 8px;
      font-size: 12px;
      font-family: "Segoe UI", "Microsoft YaHei UI", "Microsoft YaHei",
        sans-serif;
    }
    .controls {
      height: 32px;
      margin-left: auto;
      justify-content: flex-end;
      display: flex;
      .button {
        height: 100%;
        width: 46px;
        font-size: 16px;
        display: flex;
        justify-content: center;
        align-items: center;
        -webkit-app-region: no-drag;
        &:hover {
          background: var(--hover);
        }
        &:active {
          background: var(--active);
        }
        &.close {
          &:hover {
            background: rgba(232, 17, 35, 0.9);
          }
          &:active {
            background: #f1707a;
            color: #000;
          }
        }
      }
    }
  }
  &[data-theme="dark"] .win32-titlebar {
    --hover: #191919;
    --active: #333333;
  }
}

.navigation-buttons {
  flex: 1;
  display: flex;
  align-items: center;
  .svg-icon {
    height: 24px;
    width: 24px;
  }
  button {
    -webkit-app-region: no-drag;
  }
}
@media (max-width: 970px) {
  .navigation-buttons {
    flex: unset;
  }
}

.navigation-links {
  flex: 1;
  display: flex;
  justify-content: center;
  text-transform: uppercase;
  user-select: none;
  a {
    -webkit-app-region: no-drag;
    font-size: 18px;
    font-weight: 700;
    text-decoration: none;
    border-radius: 6px;
    padding: 6px 10px;
    color: var(--color-text);
    transition: 0.2s;
    -webkit-user-drag: none;
    margin: {
      right: 12px;
      left: 12px;
    }
    &:hover {
      background: var(--color-secondary-bg-for-transparent);
    }
    &:active {
      transform: scale(0.92);
      transition: 0.2s;
    }
  }
  a.active {
    color: var(--color-primary);
  }
}

.search {
  .svg-icon {
    height: 18px;
    width: 18px;
  }
}

.search-box {
  display: flex;
  justify-content: flex-end;
  -webkit-app-region: no-drag;

  .container {
    display: flex;
    align-items: center;
    height: 32px;
    background: var(--color-secondary-bg-for-transparent);
    border-radius: 8px;
    width: 200px;
  }

  .svg-icon {
    height: 15px;
    width: 15px;
    color: var(--color-text);
    opacity: 0.28;
    margin: {
      left: 8px;
      right: 4px;
    }
  }

  input {
    font-size: 16px;
    border: none;
    background: transparent;
    width: 96%;
    font-weight: 600;
    margin-top: -1px;
    color: var(--color-text);
  }

  .active {
    background: var(--color-primary-bg-for-transparent);
    input,
    .svg-icon {
      opacity: 1;
      color: var(--color-primary);
    }
  }
}

[data-theme="dark"] {
  .search-box {
    .active {
      input,
      .svg-icon {
        color: var(--color-text);
      }
    }
  }
}

.right-part {
  flex: 1;
  display: flex;
  align-items: center;
  justify-content: flex-end;
  .avatar {
    user-select: none;
    height: 30px;
    margin-left: 12px;
    vertical-align: -7px;
    border-radius: 50%;
    cursor: pointer;
    -webkit-app-region: no-drag;
    -webkit-user-drag: none;
    &:hover {
      filter: brightness(80%);
    }
  }
  .search-button {
    display: none;
    -webkit-app-region: no-drag;
  }
}
</style>
