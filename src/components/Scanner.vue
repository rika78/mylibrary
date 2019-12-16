<template>
  <div class="scanner_wrapper">
    <button @click="stop">Stop</button>
    <div ref="quagga" class="camera" />
    <pre v-if="data">
      {{ data }}
    </pre>
  </div>
</template>

<script>
import Quagga from "quagga";
import axios from "axios";
export default {
  data: () => ({ data: null, bookinfo: "" }),
  mounted() {
    this.$nextTick(() => {
      Quagga.init(
        {
          inputStream: {
            name: "Live",
            type: "LiveStream",
            target: this.$refs.quagga
          },
          decoder: {
            readers: ["ean_reader", "upc_reader"],
            debug: {
              drawBoundingBox: true
            },
            locate: true
          }
        },
        () => this.start()
      );
    });
  },
  methods: {
    start() {
      Quagga.onDetected(this.onDetected);
      Quagga.start();
      console.log("Quagga started!");
    },
    async onDetected(data) {
      this.data = data;
      var res = await axios.get(
        "https://www.googleapis.com/books/v1/volumes?q=isbn:" +
          data.codeResult.code
      );
      this.bookinfo = res.data.items[0].volumeInfo;

      // If no book is found
      // if book isn't in google api, add to database
      // If book found add to book user database
      //

      console.log(this.bookinfo);
      console.log(data);
    },
    stop() {
      Quagga.offDetected(this.onDetected);
      Quagga.stop();
      this.$refs.quagga.querySelector("video").remove();
      this.$refs.quagga.querySelector("canvas").remove();
    }
  }
};
</script>

<style>
.scanner-wrapper {
  width: 80vh;
  border-left: 1px solid grey;
  border-right: 1px solid grey;
}
.camera {
  border: 1px solid red;
  max-height: 70%;
  display: block;
}
</style>