<template>
  <div class="wrapper">
    <div class="wrapper__inner-wrapper inner-wrapper">
      <div class="container">
        <div
          class="btn red"
          :class="{ 'red-active': isLightRed }"
          @click="addPressButton(1)"
        ></div>

        <div
          class="btn green"
          :class="{ 'green-active': isLightGreen }"
          @click="addPressButton(2)"
        ></div>

        <div
          class="btn yellow"
          :class="{ 'yellow-active': isLightYellow }"
          @click="addPressButton(3)"
        ></div>

        <div
          class="btn blue"
          :class="{ 'blue-active': isLightBlue }"
          @click="addPressButton(4)"
        ></div>

        <div class="panel">
          <h1 class="title">Simon</h1>

          <div class="container-range">
            <input
              class="container-range__input input"
              type="range"
              id="level"
              min="1"
              max="3"
              v-model.number="difficultyLevel"
            />
            <label class="container-input label" for="volume">Skill level</label>
          </div>

          <div class="container-start">
            <span class="container-start__span span">Start</span>
            <button
              class="container-start__button button"
              :class="{ 'start-active': isPowerOn }"
              @click="startGame()"
            ></button>
          </div>

          <div class="display">{{ displayedInformation }}</div>

          <div class="container-power">
            <span class="container-power__span span">Power</span>

            <button
              class="container-power__button button"
              :class="{ 'power-active': isPowerOn }"
              @click="switchPower()"
            ></button>
          </div>
        </div>
      </div>
    </div>

    <div class="container-start container-start--mobile">
      <span class="container-start__span span">Start</span>
      <button
        class="container-start__button button"
        :class="{ 'start-active': isPowerOn }"
        @click="startGame()"
      ></button>
    </div>

    <div class="container-range container-range--mobile">
      <input
        class="container-range__input input"
        type="range"
        id="level"
        min="1"
        max="3"
        v-model.number="difficultyLevel"
      />
      <label class="container-input label" for="volume">Skill level</label>
    </div>

    <div class="container-power container-power--mobile">
      <span class="container-power__span span">Power</span>

      <button
        class="container-power__button button"
        :class="{ 'power-active': isPowerOn }"
        @click="switchPower()"
      ></button>
    </div>
  </div>
</template>

<script>
export default {
  props: ['sounds'],
  data() {
    return {
      roundSeries: [],
      playerSeries: [],
      isPowerOn: false,
      isRunningGame: false,
      isUserСlickAllowed: false,
      isLightRed: false,
      isLightBlue: false,
      isLightGreen: false,
      isLightYellow: false,
      isWin: false,
      isShowError: false,
      maxRound: 3,
      currentRound: 0,
      difficultyLevel: 2,
      timeBetweenSeries: 1000,
      timersId: [],
    };
  },

  computed: {
    displayedInformation() {
      let result = null;
      if (this.isShowError) {
        result = 'X';
      } else if (this.isWin) {
        result = 'WIN';
      } else if (this.currentRound) {
        result = this.currentRound;
      } else if (this.isPowerOn) {
        result = ' - - ';
      } else {
        result = '';
      }
      return result;
    },

    timeDurationLight() {
      return this.timeBetweenButtons - this.timeBetweenButtons / 4;
    },

    timeBetweenButtons() {
      let time = null;
      switch (this.difficultyLevel) {
        case 1:
          time = 1500;
          break;
        case 2:
          time = 1000;
          break;
        case 3:
          time = 400;
          break;
        default:
          break;
      }
      return time;
    },

    buttonExpectedFromUser() {
      return this.roundSeries[this.playerSeries.length - 1];
    },

    lastButtonPressedUser() {
      return this.playerSeries[this.playerSeries.length - 1];
    },
  },

  methods: {
    async startGame() {
      if (this.isPowerOn) {
        this.isRunningGame = true;
        this.resetData();
        this.newRound();
      }
    },

    async newRound() {
      this.isUserСlickAllowed = false;
      if (this.maxRound !== this.currentRound) {
        this.currentRound += 1;
        this.playerSeries = [];
        this.addTone();
        setTimeout(this.playSeries, this.timeBetweenSeries);
      } else {
        this.isWin = true;
        await this.soundWinNotification();
        this.resetData();
      }
    },

    soundWinNotification() {
      return new Promise((resolve) => {
        setTimeout(() => (
          this.playVictoryMelody({
            sound: this.lastButtonPressedUser,
            numberSounds: 6,
            resolve,
          })
        ), 1000);
      });
    },

    async playVictoryMelody({ sound, numberSounds, resolve }) {
      await this.playSound({ sound, delay: 300, speedSound: 1.5 });
      if (numberSounds > 1) {
        this.playVictoryMelody({ sound, numberSounds: numberSounds - 1, resolve });
      }
      return resolve();
    },

    switchPower() {
      this.isPowerOn = !this.isPowerOn;
      if (!this.isPowerOn) {
        this.hardReset();
      }
    },

    addTone() {
      this.roundSeries.push(this.getRandomNumber());
    },

    getRandomNumber() {
      return Math.floor(Math.random() * (5 - 1)) + 1;
    },

    addPressButton(button) {
      if (!this.isUserСlickAllowed) {
        return;
      }
      this.playerSeries.push(button);
      this.playSound({ sound: button });
    },

    notifyAboutError() {
      return new Promise((resolve) => {
        this.showError = true;
        this.playSound({ sound: this.buttonExpectedFromUser });
        setTimeout(() => resolve(), this.timeDurationLight);
      });
    },

    playSeries() {
      this.timersId = [];
      let delay = 0;
      this.roundSeries.forEach((item) => {
        delay += this.timeBetweenButtons;
        const id = setTimeout(() => {
          this.playSound({ sound: item });
        }, delay);
        this.timersId.push(id);
      });
      const totalWaitingTime = delay + this.timeDurationLight;
      const id = setTimeout(() => {
        this.isUserСlickAllowed = true;
      }, totalWaitingTime);
      this.timersId.push(id);
    },

    resetData() {
      this.roundSeries = [];
      this.playerSeries = [];
      this.currentRound = 0;
      this.isWin = false;
      this.isUserСlickAllowed = false;
    },

    playSound({ sound, delay = this.timeDurationLight, speedSound = 1 }) {
      switch (sound) {
        case 1:
          this.isLightRed = true;
          this.sounds.soundRed.pause();
          this.sounds.soundRed.currentTime = 0;
          this.sounds.soundRed.playbackRate = speedSound;
          this.sounds.soundRed.play();
          break;
        case 2:
          this.isLightGreen = true;
          this.sounds.soundGreen.pause();
          this.sounds.soundGreen.currentTime = 0;
          this.sounds.soundGreen.playbackRate = speedSound;
          this.sounds.soundGreen.play();
          break;
        case 3:
          this.isLightYellow = true;
          this.sounds.soundYellow.pause();
          this.sounds.soundYellow.currentTime = 0;
          this.sounds.soundYellow.playbackRate = speedSound;
          this.sounds.soundYellow.play();
          break;
        case 4:
          this.isLightBlue = true;
          this.sounds.soundBlue.pause();
          this.sounds.soundBlue.currentTime = 0;
          this.sounds.soundBlue.playbackRate = speedSound;
          this.sounds.soundBlue.play();
          break;
        default:
          break;
      }

      return new Promise((resolve) => {
        const id = setTimeout(() => {
          this.resetLight(sound);
          resolve();
        }, delay);
        this.timersId.push(id);
      });
    },

    hardReset() {
      this.timersId.forEach((timerId) => {
        clearTimeout(timerId);
      });
      this.isRunningGame = false;
      this.stopSounds();
      this.resetLight();
      this.resetData();
    },

    stopSounds() {
      this.sounds.soundYellow.pause();
      this.sounds.soundGreen.pause();
      this.sounds.soundBlue.pause();
      this.sounds.soundRed.pause();
      this.sounds.soundYellow.currentTime = 0;
      this.sounds.soundGreen.currentTime = 0;
      this.sounds.soundBlue.currentTime = 0;
      this.sounds.soundRed.currentTime = 0;
    },

    resetLight(sound) {
      switch (sound) {
        case 1:
          this.isLightRed = false;
          break;
        case 2:
          this.isLightGreen = false;
          break;
        case 3:
          this.isLightYellow = false;
          break;
        case 4:
          this.isLightBlue = false;
          break;
        default:
          this.isLightRed = false;
          this.isLightGreen = false;
          this.isLightYellow = false;
          this.isLightBlue = false;
          break;
      }
    },

    async checkButtonPressedCorrectly() {
      if (this.currentRound === 0) return;

      if (this.lastButtonPressedUser === this.buttonExpectedFromUser) {
        if (this.playerSeries.length === this.roundSeries.length) {
          this.newRound();
        }
      } else {
        await this.notifyAboutError();
        this.showError = false;
        this.startGame();
      }
    },
  },

  watch: {
    playerSeries() {
      this.checkButtonPressedCorrectly();
    },
  },
};
</script>
