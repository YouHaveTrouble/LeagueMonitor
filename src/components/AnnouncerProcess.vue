<template>
  <div></div>
</template>

<script>
import localforage from "localforage";

export default {
  name: "AnnouncerProcess",
  data: () => ({
    cachedEventIds: [],
  }),
  props: {
    events: {
      type: Array,
      default() {
        return [];
      }
    },
    soundVolume: {
      type: Number,
    },
    gameTime: {
      default() {
        return 0;
      }
    }
  },
  methods: {
    updateEventsList() {
      if (this.events.length <= 0) return;
      if (this.gameTime === null) return;

      // if cached list is longer than current event list, reset it
      if (this.events.length < this.cachedEventIds.length) {
        this.cachedEventIds = [];
        return;
      }

      for (const eventId in this.events) {
        const event = this.events[eventId];

        // did we already know about it?
        if (this.cachedEventIds.includes(event.EventID)) continue;
        this.cachedEventIds.push(event.EventID);

        // did it happen up to 2 seconds ago?
        if (this.gameTime - 2 > event.EventTime) continue;

        this.handleEvent(event);
      }
    },
    handleEvent(event) {
      switch (event.EventName) {
        // TODO parse friendly/enemy kill events
        // TODO queue the audio playback in case of multiple events at the same time
        default:
          localforage.getItem(event.EventName)
              .then(blob => {
                if (blob == null) return;
                const aud = new Audio(URL.createObjectURL(blob));
                aud.volume = this.soundVolume / 100;
                aud.play();
              })
              .catch(() => console.debug(`Did not find audio file to play for ${event.EventName}`));
          return;
      }
    }
  },
  mounted() {
    // skip all the events that already happened when turning the app on
    for (const eventId in this.events) {
      const event = this.events[eventId];
      this.cachedEventIds.push(event.EventID);
    }
    setInterval(() => {
      this.updateEventsList();
    }, 100);
  }
}
</script>

<style scoped>

</style>