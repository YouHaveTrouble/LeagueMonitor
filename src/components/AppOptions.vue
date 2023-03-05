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

</article>
</section>
</template>

<script>
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
      volume: window.localStorage.getItem("sound.volume") ?? "50",
    },
    announcer: {
      enabled: window.localStorage.getItem("announcer.enabled") === "true",
    }
  }),
}
</script>

<style lang="scss" scoped>
.options {
  color: #fff;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
  .option {
    display: flex;
    flex-direction: row;
    gap: 0.5rem;
    h4 {
      margin-block: 0;
    }
  }
}
</style>