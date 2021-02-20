<template>
  <div class="view" @touchstart.stop>
    <template v-if="isSplash">
      <div class="splash">
        <div class="logo">
          <img :src="require('./assets/logo.gif').default" />
        </div>
        <div class="menu">
          <div class="corner corner-tl"></div>
          <div class="corner corner-tr"></div>
          <div class="corner corner-bl"></div>
          <div class="corner corner-br"></div>
          <a class="menu-item" @click="onStartGame()">대항해시대2 플레이</a>
          <a class="menu-item" @click="onStartGame('mod_junghwa_v3.0', null, 'CHENGHO.DAT')">정화편(v3.0) 플레이</a>
          <a class="menu-item" @click="onStartGame('mod_ernst_v1.11', 'PLAY.BAT')">에르네스트 모드(v1.11) 플레이</a>
        </div>
        <div class="menu">
          <template v-if="isLoginDbx === null">
            <a class="menu-item menu-item-with-icon">
              <img :src="require('./assets/icon-dropbox.svg').default" />
              <span>인증정보를 가져오는 중</span>
            </a>
          </template>
          <template v-else-if="!isLoginDbx">
            <a class="menu-item menu-item-with-icon" @click="onLoginDropbox">
              <img :src="require('./assets/icon-dropbox.svg').default" />
              <span>로그인</span>
            </a>
          </template>
          <template v-else>
            <a class="menu-item menu-item-with-icon" @click="onLogoutDropbox">
              <img :src="require('./assets/icon-dropbox.svg').default" />
              <span>로그아웃</span>
            </a>
          </template>
        </div>
      </div>
    </template>
    <template v-else>
      <Game
        :mod="gameMod || undefined"
        :entry="gameEntry || undefined"
        :save="gameSave || undefined"
        :dbx="dbx"
      />
    </template>
  </div>
</template>
<script lang="ts">
import { Dropbox, DropboxAuth } from 'dropbox'
import Vue from 'vue'

import Game from './components/game.vue'
import { parseQueryString } from './utils'

const version = require('../package.json').version // eslint-disable-line @typescript-eslint/no-require-imports

const DBX_CLIENT_ID = '95lb7zut06xyhr8'

export default Vue.extend({
  components: {
    Game,
  },
  data() {
    return {
      isSplash: true,
      gameMod: null as string | null,
      gameEntry: null as string | null,
      gameSave: null as string | null,
      dbx: null as Dropbox | null,
      isLoginDbx: null as boolean | null,
    }
  },
  async mounted() {
    const dbxAccessToken = parseQueryString(location.hash).access_token || window.localStorage.getItem('dbx_access_token') || null as string | null
    history.replaceState({}, '', location.href.replace(location.hash, ''))

    if (dbxAccessToken) {
      const dbx = new Dropbox({ accessToken: dbxAccessToken })
      try {
        await dbx.usersGetCurrentAccount()
        window.localStorage.setItem('dbx_access_token', dbxAccessToken)
        this.dbx = dbx
        this.isLoginDbx = true
      } catch (e) {
        this.onLogoutDropbox()
      }
    } else {
      this.isLoginDbx = false
    }
  },
  computed: {
    version() {
      return version
    },
  },
  methods: {
    onStartGame(gameMod?: string, gameEntry?: string, gameSave?: string) {
      this.gameMod = gameMod ?? null
      this.gameEntry = gameEntry ?? null
      this.gameSave = gameSave ?? null
      this.isSplash = false
    },
    onLoginDropbox() {
      location.href = new DropboxAuth({ clientId: DBX_CLIENT_ID }).getAuthenticationUrl(location.href.replace(/\/+$/, ''))
    },
    onLogoutDropbox() {
      window.localStorage.removeItem('dbx_access_token')
      this.isLoginDbx = false
    },
  },
})
</script>
