<template>
  <div class="home">
    <div
      style="position: absolute; top: 5vh; width: 100%"
      class="d-flex align-center justify-center"
    >
      <div style="width: 90vw !important">
        <v-text-field solo label="Descrizione" v-model="description"></v-text-field>
      </div>
    </div>
    <v-row class="full-size d-flex align-center justify-center">
      <v-col>
        <div class="ml-15">
          <v-progress-circular
            :rotate="360"
            :size="400"
            :width="20"
            :value="valueOutOfHundred"
            color="teal"
          >
            <div>
              <h2>{{formattedHours}}:{{formattedMinutes}}:{{formattedSeconds}}</h2>
            </div>
          </v-progress-circular>
        </div>
      </v-col>
      <v-col>
        <template>
          <div>
            <TimerList :items="timerList" />
          </div>
        </template>
      </v-col>
    </v-row>
    <div
      style="position: absolute; bottom: 5vh; width: 100%"
      class="d-flex align-center justify-center"
    >
      <v-btn :disabled="isRunning" icon x-large @click="play">
        <v-icon>mdi-play-circle-outline</v-icon>
      </v-btn>
      <v-btn :disabled="!isRunning" icon x-large @click="stop">
        <v-icon>mdi-stop-circle-outline</v-icon>
      </v-btn>
      <v-btn :disabled="!isRunning" icon x-large>
        <v-icon v-on:click="copy">mdi-content-copy</v-icon>
      </v-btn>
    </div>
    <v-snackbar v-model="snackbar">{{snackbarText}}</v-snackbar>
  </div>
</template>

<script>
import moment from "moment";
import TimerList from "./TimerList.vue";

export default {
  name: "Home",
  components: {
    TimerList
  },
  data: function() {
    return {
      valueOutOfSixty: 0,
      minutes: 0,
      hours: 0,
      startAt: undefined,
      endAt: undefined,
      interval: undefined,
      description: undefined,
      summary: undefined,
      isRunning: false,
      snackbar: false,
      timerList: [],
      snackbarText: ""
    };
  },
  methods: {
    play() {
      this.isRunning = true;
      var self = this;
      this.startAt = new Date();
      this.valueOutOfSixty = 0;
      this.minutes = 0;
      this.hours = 0;

      this.interval = setInterval(() => {
        if (self.valueOutOfSixty >= 59) {
          self.valueOutOfSixty = 0;
          if (self.minutes >= 59) {
            self.minutes = 0;
            self.hours += 1;
          } else {
            self.minutes += 1;
          }
        } else {
          self.valueOutOfSixty += 1;
        }
      }, 1000);
    },
    stop() {
      if (this.checkDescription()) {
        this.timerList.push({
          text: this.getNewStamp(),
          icon: "mdi-clock"
        });
        this.isRunning = false;
        clearInterval(this.interval);
        this.endAt = new Date();
        this.valueOutOfSixty = 0;
        this.minutes = 0;
        this.hours = 0;
        this.description = "";
      }
    },
    getNewStamp() {
      var self = this;
      this.summary = "";
      this.summary += moment(this.startAt).format("YYYY/MM/DD, HH:mm");
      this.summary += " - " + moment(this.endAt).format("YYYY/MM/DD, HH:mm");
      this.summary +=
        ", " + this.formattedHours + "h " + this.formattedMinutes + "m ";
      this.summary += " - " + this.description;
      return this.summary;
    },

    copy() {
      var self = this;
      if (this.checkDescription()) {
        this.summary = this.getNewStamp();

        navigator.clipboard.writeText(this.summary).then(
          function() {
            self.snackbar = true;
          },
          function(err) {
            console.error("Async: Could not copy text: ", err);
          }
        );
      }
    },
    checkDescription() {
      if (this.description === undefined || this.description === "") {
        this.snackbarText = "Inserire una descrizione per proseguire";
        this.snackbar = true;
        return false;
      } else if (this.description !== "") {
        this.snackbarText = "Testo Copiato";
        return true;
      } else return true;
    }
  },
  computed: {
    valueOutOfHundred: function() {
      return (this.valueOutOfSixty * 100) / 60;
    },
    formattedSeconds: function() {
      return ("0" + this.valueOutOfSixty.toString()).slice(-2);
    },
    formattedMinutes: function() {
      return ("0" + this.minutes.toString()).slice(-2);
    },
    formattedHours: function() {
      return ("0" + this.hours.toString()).slice(-2);
    }
  }
};
</script>

<style scoped>
.full-size {
  height: 100vh;
  width: 100vw;
}
</style>
