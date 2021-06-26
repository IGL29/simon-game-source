<template>
 <div>
    <audio
      ref="soundRed"
      src="https://s3.amazonaws.com/freecodecamp/simonSound1.mp3"
      preload="auto"
    ></audio>

    <audio
      ref="soundGreen"
      src="https://s3.amazonaws.com/freecodecamp/simonSound2.mp3"
      preload="auto"
    ></audio>

    <audio
      ref="soundYellow"
      src="https://s3.amazonaws.com/freecodecamp/simonSound3.mp3"
      preload="auto"
    ></audio>

    <audio
      ref="soundBlue"
      src="https://s3.amazonaws.com/freecodecamp/simonSound4.mp3"
      preload="auto"
    ></audio>

   <SimonGame :sounds="$refs" />
   <ButtonMuteSound :isMuteSound.sync="isMuteSound"/>
 </div>
</template>

<script>
import SimonGame from '@/components/SimonGame.vue';
import ButtonMuteSound from '@/components/ButtonMuteSound.vue';

export default {
  data() {
    return {
      isMuteSound: false,
    };
  },

  watch: {
    isMuteSound() {
      Object.values(this.$refs).forEach((value) => {
        const sound = value;
        sound.muted = this.isMuteSound;
      });
    },
  },

  components: { SimonGame, ButtonMuteSound },
};
</script>
