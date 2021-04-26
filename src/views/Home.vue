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
    <v-row class="full-size d-flex align-center justify-center mt-15">
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
      snackbarText: "",
      startDate: undefined,
      endDate: undefined,
      resultDate: undefined,
      prevDescription: ""
    };
  },
  methods: {
    play() {
      this.startDate = this.currentDate();
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
      this.endDate = this.currentDate();
      if (this.checkDescription()) {
        var activity = this.getNewStamp();
        this.timerList.push({
          text: activity,
          icon: "mdi-clock"
        });
        this.isRunning = false;
        clearInterval(this.interval);
        this.endAt = new Date();
        this.valueOutOfSixty = 0;
        this.minutes = 0;
        this.hours = 0;
        this.description = "";
        this.copy(activity);
      }
    },
    getNewStamp() {
      this.resultDate = this.dateDiff();
      this.resultDate += " - " + this.description;
      this.prevDescription = this.resultDate;
      return this.resultDate;
    },

    copy(activity) {
      var self = this;
      navigator.clipboard.writeText(activity).then(
        function() {
          self.snackbar = true;
          self.snackbarText = "Attività conclusa. Testo Copiato";
        },
        function(err) {
          console.error("Async: Could not copy text: ", err);
        }
      );
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
    },
    currentDate() {
      var myDate = new Date();
      var year = myDate.getFullYear();
      var month = myDate.getMonth();
      if (month <= 9) month = "0" + (month + 1);
      var day = myDate.getDate();
      if (day <= 9) day = "0" + day;
      var hour = myDate.getHours();
      if (hour <= 9) hour = "0" + hour;
      var minutes = myDate.getMinutes();
      if (minutes <= 9) minutes = "0" + minutes;
      var seconds = myDate.getSeconds();
      if (seconds <= 9) seconds = "0" + seconds;

      var date = new Date(year, month, day, hour, minutes, seconds);

      return date;
    },

    dateDiff() {
      if (this.endDate < this.startDate) {
        this.endDate.setDate(this.endDate.getDate() + 1);
      }
      var diff = this.endDate - this.startDate;

      var msec = diff;
      var hh = Math.floor(msec / 1000 / 60 / 60);
      msec -= hh * 1000 * 60 * 60;
      var mm = Math.floor(msec / 1000 / 60);
      msec -= mm * 1000 * 60;
      var ss = Math.floor(msec / 1000);
      msec -= ss * 1000;

      var result = "";
      result += "L'ultima attività è durata :  ";
      if (hh != 0) {
        result += hh;
        result += "h, ";
      }

      if (mm != 0) {
        result += mm;
        result += "m e ";
      }
      result += ss;
      result += "s";

      return result;
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
