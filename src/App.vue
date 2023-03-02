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
  <main v-if="itemData !== null">
    <keep-alive>
      <LiveGame :game-data="gameData" :item-data="itemData" v-if="currentTab === 'liveGame'"/>
    </keep-alive>
  </main>
</template>

<script>

import LiveGame from "./components/LiveGame.vue";

const request = require('request');

export default {
  name: 'App',
  components: {
    LiveGame,
  },
  data() {
    return {
      currentTab: "liveGame",
      gameCheck: null,
      gameData: null,
      itemData: null,
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
        if (error) {
          this.gameData = null;
        } else {
          this.gameData = JSON.parse(body);
        }
      });
      this.gameCheck = setTimeout(() => {
        this.checkForGame();
      }, 100);
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

    }
  },
  async mounted() {
    await this.getItemData();
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
