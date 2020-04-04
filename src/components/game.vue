<template lang="html">
  <section class="game">
    <div class="md-layout md-gutter">
      <div class="md-layout-item md-size-50 md-medium-size-100">
        <video
          muted
          src=""
          id="video"
          style="width:100%; height: 100%;"
          autoplay="true"
        ></video>
      </div>
      <div class="md-layout-item md-size-50 md-medium-size-100">
        <img id="play" />
      </div>
      <canvas style="display:none" id="preview"></canvas>
    </div>
    <button @click="clickButton('hola')">Saludar</button>
  </section>
</template>

<script lang="js">

  import io from 'socket.io-client';

  export default  {
    name: 'game',
    props: [],
    mounted () {

      var canvas = document.getElementById("preview");
      var context = canvas.getContext('2d');

      canvas.width = 400;
      canvas.height = 300;

      context.width = canvas.width;
      context.height = canvas.height;

      var video = document.getElementById("video");

      navigator.getUserMedia = ( navigator.getUserMedia || navigator.webkitGetUserMedia || navigator.mozGetUserMedia || navigator.msgGetUserMedia );

      if(navigator.getUserMedia)
      {
          navigator.getUserMedia({
              video: true,
              audio: true
          },this.loadCamera,this.loadFail);
      }
      setInterval(()=>{
          this.draw(video,context,canvas);
      },0.1);

      this.socket.on('connect', function () {
          console.log('socket connected')
      });
      this.socket.on('stream', function(image){
          document.getElementById('play').setAttribute('src',image);
      });
    },
    data () {
      return {
        texto: "Hola",
        socket: io(),
      }
    },
    methods: {
      clickButton(texto){
        this.socket.emit('mensaje', texto)
      },
      draw(video,context,captura){
          context.drawImage(video,0,0,context.width,context.height);
          this.socket.emit('stream',captura.toDataURL('image/webp'));
      },
      loadFail(){
          console.log("Camera not connected");
      },
      loadCamera(stream){
        try {
            var video = document.getElementById("video");
            console.log(stream);
            console.log(stream.getVideoTracks());
            video.srcObject = stream;
        }

        catch (error) {
          video.src = URL.createObjectURL(stream);
        }
          console.log("Camera connected");
      }
    },
    computed: {

    }
}
</script>

<style scoped lang="css">
#video,
#play {
  border: 2px solid black;
  height: 400px;
}
img {
  min-width: 100%;
  min-height: 100%;
}
</style>
