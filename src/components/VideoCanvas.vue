<template>
  
  <div id="container"
        ref="container"
        class="container">

    <div>
      Video Canvas with Slider
    </div>

    <!-- <div id="infoDiv"
        ref="infoDiv">

      <p>{{ message }}</p>
    </div> -->

    <div>
      <input type="button"  @click="toggleVideo" value="Show / Hide Video" />
    </div>

    <div>
      <video id="video"
              ref="video"
              :height="height"
              :width="width"
              :style="`display: ${ showVideo ? 'unset' : 'none' } ;`"
              src="https://os.zhdk.cloud.switch.ch/envicloud/wsl/CORE_S2A/3_core_stream/CORE_h264_ovr_0.mp4"
              controls="true"
              crossorigin="anonymous"/>
              <!-- src="/CORE_h264_ovr_23.mp4" -->
              
              <!-- src="https://os.zhdk.cloud.switch.ch/envicloud/wsl/CORE_S2A/3_core_stream/CORE_h264_ovr_23.mp4" -->

      <!-- <video id="video"
              ref="video"
              :height="height"
              :width="width"
              class="video-js">
        <source src="/CORE_h264_ovr_23.mp4" type="video/mp4">
      </video>               -->

    </div>

    <div>
      <input type="range"
              id="slider"
              name="Frame Slider"
              :style="`width: ${width}px; `"
              :min="minFrame"
              :max="maxFrame"
              step="1"
              :value="frameSlider"
              @change="frameSliderChanged" />
      <div>{{ `Frame: ${frameSlider}` }}</div>
    </div>

    <!-- <div>
      <input type="range"
              id="widthSlider"
              name="Width Slider"
              :style="`width: ${width}px; `"
              min="100"
              :max="$refs.video ? $refs.video.width : 1000"
              step="10"
              :value="500"
              @change="(e) => { cWidth = e.target.value; }" />
      <div>{{ `Width: ${cWidth}` }}</div>
    </div> -->

    <div style="margin: 50px auto 0px auto; width: 300px; display: grid; grid-template-columns: 2fr 1fr 2fr; grid-gap: 5px;">
      <div >
        Picked <br/> Pixel
      </div>

      <div :style="`aspect-ratio: 1; border: 1px black solid; background-color: ${pixelColor}; `">

      </div>
      <div >
        {{ pixelColor }}
      </div>
    </div>

    <div style="margin: 50px auto 0px auto; display: inline-block; border: 1px solid grey;">
      <div v-for="(g, gi) in pixelGrid"
            :key="`row_${gi}`"
            style="margin: 0; display: flex; flex-direction: row;">

        <div v-for="(p, i) in g"
              :key="i"
              :style="`display: flex; flex-direction: column;
                        width: 10px; height: 10px;
                        background-color: ${pixelColorStyles(p)}`"
                        @click="pickPixel" >
                        <!-- border: 1px solid grey; -->

        </div>
      </div>
    </div>

    <div>
      <input type="range"
              id="pixelSlider"
              name="Pixels Slider"
              :style="`width: ${width}px; `"
              min="3"
              max="15"
              step="2"
              :value="3"
              @change="(e) => { pixelAmount = Number.parseInt(e.target.value); }" />
      <div>{{ `Pixels: ${pixelAmount}` }}</div>
    </div>

    <div>
      <canvas id="c2"
              ref="c2"
              :height="cHeight"
              :width="cWidth"
              @click="canvasClick"
              style="border: 1px black solid; "></canvas>
              <!-- @mousemove="canvasClick" -->

    </div>

  </div>

  
</template>

<script >
import { ref } from 'vue';

export default {
  props: {
  },
  setup(props) {
    let message = ref('Info about the video')
    const height = ref(400);
    const width = ref(400);

    const cHeight = ref(1024);
    const cWidth = ref(1024);
    const lastX = ref(null);
    const lastY = ref(null);
    const maxHeight = ref(500);
    const maxWidth = ref(500);
    const player = ref(null);

    const minFrame = ref(0);
    const maxFrame = ref(100);
    const frameSlider = ref(0);

    const showVideo = ref(true);
    const pixelColor = ref(null);
    const pixelAmount = ref(3);
    const pixels = ref([]);

    return {
      message,
      height,
      cHeight,
      maxHeight,
      width,
      cWidth,
      lastX,
      lastY,
      maxWidth,
      player,
      minFrame,
      maxFrame,
      frameSlider,
      showVideo,
      pixelColor,
      pixelAmount,
      pixels,
    }
  },
  mounted() {

    if (!('requestVideoFrameCallback' in HTMLVideoElement.prototype)) {
      this.message = 'requestVideoFrameCallback() method is not support by this browser, use chorme instead!';
      return;
    }

    const vid = this.$refs.video;
    vid.onloadeddata = (event) => {
      this.maxFrame = event.target.duration;

      this.maxWidth = event.target.width;
      this.maxHeight = event.target.height;
      // console.log(vid.duration);
    };

    // vid.oncanplay = (event) => {
    //   console.log('Video can start, but not sure it will play through.');
    //   console.log(event);
    // };

    // vid.onloadedmetadata = (event) => {
    //   console.log('The duration and dimensions of the media and tracks are now known. ');
    //   console.log(event.target);
    // };    

    const canvas = this.$refs.c2.getContext('2d');

    // let startTime = 0.0;

    const updateCanvas = (now, metadata) => {
      // if (startTime === 0.0) {
      //   startTime = now;
      // }

      // const frame = canvas.getImageData(0, 0, this.width, this.height);
      canvas.drawImage(vid, 0, 0, this.cWidth, this.cHeight);
      // canvas.drawImage(frame, 0, 0);
      // this.frames.push(frame);

      // const elapsed = (now - startTime) / 1000.0;
      // const fps = (++paintCount / elapsed).toFixed(3);
      // fpsInfo.innerText = `video fps: ${fps}`;
      // this.message = JSON.stringify(metadata, null, 2);

      vid.requestVideoFrameCallback(updateCanvas);
    };

    vid.requestVideoFrameCallback(updateCanvas);    

    //   const stream = vid.captureStream(25);
    //   const videoTrack = stream.getVideoTracks()[0];

    //   if (videoTrack) {

    //     // console.log(info);
    //     // console.log(info.getVideoTracks().length);
    //     // console.log(info.getAudioTracks().length);

    //     canvas.drawImage(vid, 0, 0, vid.videoWidth, this.height);        

    //     // this.message = videoTrack.requestFrame();
    //   }

    // }, false);

  },
  beforeDestroy() {
    if (this.player) {
        this.player.dispose()
    }
  },
  computed: {
    pixelColorStyle() {
      return this.pixelColor ? `rgba(${this.pixelColor.r}, ${this.pixelColor.g}, ${this.pixelColor.b}, ${this.pixelColor.a});` : '';
    },
    pixelGrid() {
      const grid = [];

      for (var i = 0, n = this.pixels.length; i < n; i += this.pixelAmount) {
        const rowOfPixels = this.pixels.slice(i, i + this.pixelAmount);
        grid.push(rowOfPixels);
      }

      return grid;
    },
  },
  watch: {
    pixelAmount () {
      this.canvasClick({});
    },
  },
  methods: {
    frameSliderChanged(event) {
      const newFrame = event.target.value;
      this.frameSlider = newFrame;
      // console.log(newFrame);

      const vid = this.$refs.video;
      vid.currentTime = newFrame;
      const canvas = this.$refs.c2.getContext('2d');
      canvas.drawImage(vid, 0, 0, this.cWidth, this.cHeight);
      
      this.$nextTick(() => {
        this.canvasClick(event);
      });
    },
    toggleVideo(e) {
      this.showVideo = !this.showVideo;
    },
    canvasClick(e) {
      const px = e.layerX ? (e.layerX - this.pixelAmount) : this.lastX;
      const py = e.layerY ? (e.layerY - this.pixelAmount) : this.lastY;

      if (!px || !py) {
        return;
      }

      this.lastX = px;
      this.lastY = py;
      const pWidth = this.pixelAmount;

      const pixel = this.$refs.c2.getContext('2d').getImageData(this.lastX, this.lastY, pWidth, pWidth);
      const rgbData = pixel.data;
      // this.pixelColor = this.getPixelRGB(rgbData);

      this.pixels = [];

      for (var i = 0, n = rgbData.length; i < n; i += 4) {
        const pixelValues = rgbData.slice(i, i + 4);
        // pix[i  ] = 255 - pix[i  ]; // red
        // pix[i+1] = 255 - pix[i+1]; // green
        // pix[i+2] = 255 - pix[i+2]; // blue
        // // i+3 is alpha (the fourth element)
        const currentPix = this.getPixelRGB(pixelValues);
        this.pixels.push(currentPix);
      }
    },
    getPixelRGB(rgbData) {
      const r = rgbData[0];
      const g = rgbData[1];
      const b = rgbData[2];
      const a = rgbData[3];

      return {r, g, b, a};
    },
    pixelColorStyles(pixel) {
      return pixel ? `rgba(${pixel.r}, ${pixel.g}, ${pixel.b}, ${pixel.a});` : '';
    },
    pickPixel(e) {
      const divStyles = e.target.style;
      this.pixelColor = divStyles.backgroundColor || '';
    },
  },
}
</script>



<style scoped>

.container > * + * {
  margin-top: 1.5rem;
}
</style>
