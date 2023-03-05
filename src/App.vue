<template>
  <div class="loader" v-if="itemData === null"></div>
  <nav v-if="itemData !== null">
    <div class="navItem" @click="currentTab = 'liveGame'" :class="{active: currentTab === 'liveGame'}">
      <span>Live game <span class="dot" v-if="gameData !== null"></span></span>
    </div>
    <div class="navItem" @click="currentTab = 'options'" :class="{active: currentTab === 'options'}">
      <span>Options</span>
    </div>
  </nav>
  <main v-if="itemData !== null && championData !== null">
    <keep-alive>
      <LiveGame
          :game-data="gameData"
          :item-data="itemData"
          :champion-data="championData"
          v-if="currentTab === 'liveGame'"
      />
    </keep-alive>
    <AppOptions
        v-if="currentTab === 'options'"
        @volumeChange="options.sound.volume = $event"
        @announcerEnabledChange="options.announcer.enabled = $event"
    />
  </main>
</template>

<script>

import LiveGame from "./components/LiveGame.vue";
import AppOptions from "@/components/AppOptions.vue";

const request = require('request');
const { version, displayName } = require('../package.json');

export default {
  name: 'App',
  components: {
    AppOptions,
    LiveGame,
  },
  data() {
    return {
      currentTab: "liveGame",
      gameCheck: null,
      gameData: null,
      itemData: null,
      championData: null,
      supportedGamemodes: [
          "CLASSIC",
          "ARAM",
          "PRACTICETOOL",
      ],
      options: {
        sound: {
          volume: window.localStorage.getItem("sound.volume") ? Number.parseInt(window.localStorage.getItem("sound.volume")) : 50,
        },
        announcer: {
          enabled: window.localStorage.getItem("announcer.enabled") === "true",
        }
      }
    }
  },
  methods: {
    async checkForGame() {
      await request.get({
        url: "https://127.0.0.1:2999/liveclientdata/allgamedata",
        rejectUnauthorized: false,
        requestCert: false,
        agent: false,
      }, (error, response, body) => {
        this.gameCheck = setTimeout(() => {
          this.checkForGame();
        }, 200);
        if (error) {
          this.gameData = null;
        } else {
          if (response.statusCode !== 200) {
            this.gameData = null;
            return;
          }
          const dataJson = JSON.parse(body);

          if (!this.supportedGamemodes.includes(dataJson.gameData.gamemode)) {
            this.gameData = dataJson;
          }
        }
      });


    },
    async getItemData() {
      await request.get({
        url: "https://ddragon.leagueoflegends.com/cdn/13.4.1/data/en_US/item.json",
      }, (error, response, data) => {
       if (error) {
         return;
       }
       const jsonItemData = JSON.parse(data);
       this.itemData = jsonItemData.data;
      });

    },
    async getChampionData() {
      await request.get({
        url: "https://ddragon.leagueoflegends.com/cdn/13.4.1/data/en_US/champion.json",
      }, (error, response, data) => {
        if (error) {
          return;
        }
        const jsonChampionData = JSON.parse(data);

        this.championData = {};
        for (const champId in jsonChampionData.data) {
          this.championData[champId.toLocaleLowerCase("EN")] = jsonChampionData.data[champId];
        }
      });

    }
  },
  async mounted() {
    document.title = `${displayName} ${version}`

    await this.getItemData();
    await this.getChampionData();
    await this.checkForGame();
  },
  beforeUnmount() {
    clearTimeout(this.gameCheck);
    this.gameCheck = null;
  }
}
</script>

<style lang="scss">
nav {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  align-items: center;
  background-color: #2c2c2c;
  color: #fff;
  .navItem {
    padding: 0.2rem 1.2rem;
    position: relative;
    cursor: pointer;
    &.active {
      background-color: rgba(255,255,255, 0.2);
      cursor: default;
    }
    &:hover:not(.active) {
      background-color: rgba(255,255,255, 0.1);
    }
    .dot {
      position: absolute;
      background-color: limegreen;
      border-radius: 50%;
      width: 0.5rem;
      height: 0.5rem;
      top: 50%;
      transform: translateY(-60%);
      left: 0.4rem;
    }
  }
}
main {
  position: relative;
  width: 100%;
}
.loader {
  display: inline-flex;
  justify-content: center;
  align-items: center;
  flex-direction: column-reverse;
  width: 100%;
  &:after {
    content: " ";
    display: block;
    width: 64px;
    height: 64px;
    margin: 8px;
    border-radius: 50%;
    border: 6px solid #fff;
    border-color: #fff transparent #fff transparent;
    animation: lds-dual-ring 1.2s linear infinite;
  }
}
@keyframes lds-dual-ring {
  0% {
    transform: rotate(0deg);
  }
  100% {
    transform: rotate(360deg);
  }
}
</style>
