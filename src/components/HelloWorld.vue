<template>
  <div>
    <div class="hello" @click="on_start">{{ msg }}</div>
    <div class="ind">出力先選択</div>
    <button 
      v-for="out_dev in out_infos"
      :key=out_dev.deviceId
      @click="select_out(out_dev)"
    >{{ out_dev.label }}</button>
    <div class="ind">入力先選択</div>
    <button 
      v-for="in_dev in in_infos"
      :key=in_dev.deviceId
      @click="select_in(in_dev)"
    >{{ in_dev.label }}</button>
  </div>
</template>

<script>
export default {
  name: 'HelloWorld',
  data() {
    return {
      msg: "",
      audio: null,
      audioCtx: null,
      stream: null,
      out_infos: [],
      in_infos: []
    }
  },
  mounted() {
    this.msg = "開始";
    
  },
  methods: {
    async on_start() {
      if (this.msg == "開始") {
        this.msg = "終了";
        try {
          this.audioCtx = new AudioContext();
          this.audio = new Audio();

          this.stream = await navigator.mediaDevices.getUserMedia({ video: false, audio: true });

          const devices = await navigator.mediaDevices.enumerateDevices();
          this.out_infos = devices.filter(info => {
            return info.kind == "audiooutput" && info.deviceId != "default" && info.deviceId != "communications" 
          });
          console.dir(this.out_infos);
          this.in_infos = devices.filter(info => {
            return info.kind == "audioinput" && info.deviceId != "default" && info.deviceId != "communications" 
          });
          console.dir(this.in_infos);

          const src = this.audioCtx.createMediaStreamSource(this.stream);
          const dst = this.audioCtx.createMediaStreamDestination();
          src.connect(dst);
          this.audio.srcObject = dst.stream;
          await this.audio.play();

        } catch (err) {
          alert(err);
        }
      } else {
        this.msg = "開始";
        this.stream.getAudioTracks()[0].stop();
        this.stream = null;
      }
    },
    async select_out(dev) {
      await this.audio.setSinkId(dev.deviceId);
    },
    async select_in(dev) {
      console.log(dev);
      try {
        this.audio.pause();
        this.audio.srcObject = null;
        this.stream = await navigator.mediaDevices.getUserMedia({
          video: false,
          audio: {
            deviceId: dev.deviceId
          }
        });
        const src = this.audioCtx.createMediaStreamSource(this.stream);
        const dst = this.audioCtx.createMediaStreamDestination();
        src.connect(dst);
        this.audio.srcObject = dst.stream;
        await this.audio.play();
      } catch (err) {
        alert(err);
      }
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.hello {
  font-size: 5rem;
  margin: 40px 0 0;
}
.ind {
  margin-top: 10px;
  font-size: 2rem;
}
button {
  font-size: 1rem;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  width: 80%;
  margin: 5px;
}
</style>
