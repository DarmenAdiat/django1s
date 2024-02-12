<template>
  <v-app>
    <Navbar />
    <v-main class='ma-4'>
      <div :className="className" :style="containerStyle">
        <canvas :id="id" ref="spline" :style="canvasStyle"></canvas>
      </div>
      <router-view></router-view>
    </v-main>
    <Footer />
  </v-app>
</template>
<script>
import "~/three";

import { speRuntimeFactory } from "~/spline.runtime.min";
const SpeRuntime = speRuntimeFactory({}, window.THREE);
import Navbar from "@/components/Navbar";
import Footer from "@/components/Footer";

export default {
  props: {
    id: {
      type: String,
    },
    scene: {
      type: Object,
    },
    className: {
      type: String,
    },
    canvasStyle: {
      type: Object,
      default: () => {
        return { height: "100%", width: "100%", outline: "none" };
      },
    },
    containerStyle: {
      type: Object,
      default: () => {
        return {
          width: "100%",
          height: "100%",
          overflow: "hidden",
          position: "relative",
        };
      },
    },
  },
  mounted() {
    new SpeRuntime.Application().start(this.scene, this.$refs.spline)
  },
  components: { Footer, Navbar},
}
</script>
