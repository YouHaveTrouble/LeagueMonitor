<template>
<section v-if="gameData !== null" class="live-game">

  <article class="currentPlayer">
    <div class="champion-icon">
      <img
          v-if="activePlayerChampion"
          :src="`https://ddragon.leagueoflegends.com/cdn/13.4.1/img/champion/${encodeURI(championData[activePlayerChampion.championName]?.image.full)}`"
          alt=""
          draggable="false"
      >
      <div class="champion-level">{{ activePlayerChampion.level }}</div>
    </div>
    <div class="gold" v-if="activePlayer.currentGold !== null">{{ Math.round(activePlayer.currentGold) }}</div>
    <div class="gold">Gold in items: {{ goldInItems }}</div>
  </article>
  <article class="enemies">
    <div class="player" v-for="player in enemies" :key="player.summonerName">
      <div class="champion-icon">
        <img
            :src="`https://ddragon.leagueoflegends.com/cdn/13.4.1/img/champion/${encodeURI(championData[player.championName]?.image.full)}`"
            alt=""
            draggable="false"
        >
        <div class="champion-level">{{ player.level }}</div>
      </div>
      <div class="gold">Gold in items: {{ player.goldInItems }}</div>
      <div class="gold">Gold {{ goldInItems -player.goldInItems  >= 0 ? 'advantage' : 'disadvantage' }}: {{ Math.abs(player.goldInItems - goldInItems) }}</div>
    </div>
  </article>

</section>
</template>

<script>
export default {
  name: "LiveGame",
  data() {
    return {
      activePlayer: {},
      activePlayerChampion: {},
      events: [],
      goldInItems: 0,
      enemies: [],
    }
  },
  props: {
    gameData: {
      type: Object,
    },
    itemData: {
      type: Object,
    },
    championData: {
      Type: Object,
    }
  },
  methods: {
    updateCurrentPlayerData() {
      if (this.gameData === null) return;
       this.activePlayer = this.gameData.activePlayer;
       for (const id in this.gameData.allPlayers) {
         const champion = this.gameData.allPlayers[id];
         if (champion.summonerName === this.activePlayer.summonerName) {
           this.activePlayerChampion = champion;
           this.activePlayerChampion.championName = this.normalizeChampionName(this.activePlayerChampion.championName);
           break;
         }
       }
       this.goldInItems = this.calculateItemsWorth(this.activePlayerChampion.items)
    },
    updateEnemies() {
      this.enemies = [];
      if (this.gameData === null) return;
      for (const id in this.gameData.allPlayers) {

        const player = {};
        const champion = this.gameData.allPlayers[id];

        if (champion.team === this.activePlayerChampion.team) continue;

        player.goldInItems = this.calculateItemsWorth(champion.items);
        player.summonerName = champion.summonerName;
        player.championName = this.normalizeChampionName(champion.championName);
        player.level = champion.level;

        this.enemies.push(player);
      }
    },
    calculateItemsWorth(items) {
      let worth = 0;

      for (const itemIndex in items) {
        const item = items[itemIndex];
        if (item.consumable) continue;
        const itemId = item.itemID;
        worth += this.itemData[itemId]?.gold?.total;
      }

      return worth;
    },
    normalizeChampionName(name) {

      name = name.replace("'", "");
      name = name.replace(" ", "");

      return name.toLowerCase();
    }
  },
  mounted() {
    this.updateCurrentPlayerData();
    this.updateEnemies();
    setInterval(() => {
      this.updateCurrentPlayerData();
      this.updateEnemies();
    }, 100);
  }
};
</script>

<style lang="scss" scoped>
.live-game {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-direction: column;
  width: 100%;
  .currentPlayer {
    padding-block: 1rem;
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
    flex-direction: column;
    gap: 0.75rem;
  }
  .enemies {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;
    padding-top: 3rem;
    width: 100%;
    gap: 1.5rem;
    .player {
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      gap: 0.75rem;
    }
  }
  .champion-icon {
    position: relative;
    display: flex;
    justify-content: center;
    align-items: center;
    width: 6rem;
    height: 6rem;
    .champion-level {
      position: absolute;
      display: flex;
      justify-content: center;
      align-items: center;
      font-size: 1.1rem;
      color: #fff;
      font-weight: bold;
      background-color: #2c2c2c;
      border: 2px solid #000;
      width: 2rem;
      height: 2rem;
      bottom: -0.5rem;
      right: -0.5rem;
      border-radius: 0.25rem;
    }
    img {
      width: 100%;
      height: 100%;
      object-fit: cover;
      object-position: center;
    }
  }
  .gold {
    padding-inline: 0.5rem;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #efc456;
    font-weight: bold;
  }
}

</style>