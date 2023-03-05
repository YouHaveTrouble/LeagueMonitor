<template>
  <section class="options">
    <article class="sound">
      <h3>Sound</h3>
      <div class="option">
        <h4>Volume</h4>
        <input type="range" min="0" max="100" v-model="sound.volume">
        <span>{{ sound.volume }}%</span>
      </div>
      <div class="option">
        <h4>Announcer</h4>
        <input type="checkbox" v-model="announcer.enabled">
      </div>
      <div class="files">
        <h3>Announcer</h3>
        <div class="option">
          <span>Game Start</span>
          <input type="file" ref="GameStart" accept=".mp3,.wav,.opus,.ogg,.aac">
          <input type="button" value="Add" @click="handleAudioFile('GameStart')">
        </div>
        <div class="option">
          <span>First Blood</span>
          <input type="file" ref="FirstBlood" accept=".mp3,.wav,.opus,.ogg,.aac">
          <input type="button" value="Add" @click="handleAudioFile('FirstBlood')">
        </div>

      </div>

    </article>
  </section>
</template>

<script>
import localforage from "localforage";

export default {
  name: "AppOptions",
  watch: {
    sound: {
      deep: true,
      handler(newSoundOptions) {
        window.localStorage.setItem("sound.volume", newSoundOptions.volume);
        this.$emit("volumeChange", Number.parseInt(newSoundOptions.volume))
      }
    },
    announcer: {
      deep: true,
      handler(newValue) {
        window.localStorage.setItem("announcer.enabled", `${newValue.enabled}`)
        this.$emit("announcerEnabledChange", newValue.enabled);
      }
    }
  },
  data: () => ({
    sound: {
      volume: window.localStorage.getItem("sound.volume") ?? 50,
    },
    announcer: {
      enabled: window.localStorage.getItem("announcer.enabled") === "true",
    },
    soundFiles: {},
  }),
  methods: {
    async handleAudioFile(eventname) {
      const file = this.$refs[eventname].files[0];
      await localforage.removeItem(eventname);
      await localforage.setItem(eventname, file);
    }
  },
}
</script>

<style lang="scss" scoped>
.options {
  color: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;

  .row {
    display: flex;
    flex-direction: row;
    justify-content: space-between;
    padding-block: 0.25rem;
    gap: 0.5rem;
  }

  .option {
    display: flex;
    flex-direction: row;
    gap: 0.5rem;

    input[type="file"] {
      text-align: end;
    }

    h4 {
      margin-block: 0;
    }
  }
}
</style>