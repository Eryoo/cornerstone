<template>
    <div>
        <div class="image-canvas-wrapper" oncontextmenu="return false">
          <div id="element-1"></div>
            <!-- DICOM CANVAS -->
            <span id="loadProgress">Diocm加载: </span>
            <div ref="canvas" id="canvas" class="image-canvas"></div>
        </div>
    </div>
</template>

<script>
//引入 cornerstone,dicomParser,cornerstoneWADOImageLoader
import * as cornerstone from "cornerstone-core";
import * as dicomParser from "dicom-parser";

import Hammer from "hammerjs";
// 不建议 npm 安装 cornerstoneWADOImageLoader 如果你做了 会很头疼
import * as cornerstoneWADOImageLoader from "../../static/dist/cornerstoneWADOImageLoader.min.js";
// Cornerstone 工具外部依赖

import * as cornerstoneMath from "cornerstone-math";
import * as cornerstoneTools from "cornerstone-tools";

// Specify external dependencies
cornerstoneTools.external.cornerstoneMath = cornerstoneMath;
cornerstoneTools.external.cornerstone = cornerstone;
cornerstoneTools.external.Hammer = Hammer;
cornerstoneTools.init({
  /**
   * When cornerstone elements are enabled,
   * should `mouse` input events be listened for?
   */
  mouseEnabled: true,
  /**
   * When cornerstone elements are enabled,
   * should `touch` input events be listened for?
   */
  touchEnabled: true,
  /**
   * A special flag that synchronizes newly enabled cornerstone elements. When
   * enabled, their active tools are set to reflect tools that have been
   * activated with `setToolActive`.
   */
  globalToolSyncEnabled: false,
  /**
   * Most tools have an associated canvas or SVG cursor. Enabling this flag
   * causes the cursor to be shown when the tool is active, bound to left
   * click, and the user is hovering the enabledElement.
   */
  showSVGCursors: true,
});
cornerstoneWADOImageLoader.external.cornerstoneMath = cornerstoneMath;
cornerstoneWADOImageLoader.external.dicomParser = dicomParser;
//指定要注册加载程序的基石实例
cornerstoneWADOImageLoader.external.cornerstone = cornerstone;

cornerstone.registerImageLoader("http", cornerstoneWADOImageLoader.loadImage);
cornerstone.registerImageLoader("https", cornerstoneWADOImageLoader.loadImage);

//配置 webWorker (必须配置)
//注意这里的路径问题  如果路径不对 cornerstoneWADOImageLoaderWebWorker 会报错 index.html Uncaught SyntaxError: Unexpected token <
var config = {
  webWorkerPath: "/static/dist/cornerstoneWADOImageLoaderWebWorker.js",
  taskConfiguration: {
    decodeTask: {
      codecsPath: "/static/dist/cornerstoneWADOImageLoaderCodecs.js",
    },
  },
};

cornerstoneWADOImageLoader.webWorkerManager.initialize(config);
import axios from "axios";
export default {
  name: "HelloWorld",
  data() {
    return {
      baseUrl: "",
      exampleStudyImageIds: "",
      isInitLoad: true,
      isShow: true,
    };
  },
  methods: {
    initCanvasTools() {
      let _self = this;
      let canvas = this.$refs.canvas;
      canvas = document.getElementById("canvas");
      cornerstone.enable(canvas);
      const StackScrollTool = cornerstoneTools.StackScrollTool;

      const imageIds = [];
      for (let i = 1; i < 38; i++) {
        imageIds.push(
          "wadouri:" +
            `https://tools.cornerstonejs.org/examples/assets/dicom/bellona/chest_lung/${i}.dcm`
        );
      }

      const stack = {
        currentImageIdIndex: 0,
        imageIds,
      };
      canvas.tabIndex = 0;
      canvas.focus();
      console.log(cornerstoneTools);

      cornerstone.loadImage(imageIds[0]).then((image) => {
        cornerstone.displayImage(canvas, image);
        cornerstoneTools.addStackStateManager(canvas, ["stack"]);
        cornerstoneTools.addToolState(canvas, "stack", stack);
      });

      cornerstoneTools.addTool(StackScrollTool);
      cornerstoneTools.setToolActive("StackScroll", { mouseButtonMask: 1 });
      cornerstoneTools.init();
      const LengthTool = cornerstoneTools.LengthTool;

      // Make sure we have at least one element Enabled
      const element = document.querySelector('#element-1');
      cornerstone.enable(element);

      // Adds tool to ALL currently Enabled elements
      cornerstoneTools.addTool(LengthTool);

      // OR add the tool to a specific Enabled element
      cornerstoneTools.addToolForElement(element, LengthTool);
    },
    /*
     * Window Resize
     *
     */
    listenForWindowResize() {
      this.$nextTick(function () {
        window.addEventListener(
          "resize",
          this.debounce(this.onWindowResize, 100)
        );
      });
    },
    onWindowResize() {
      cornerstone.resize(this.$refs.canvas, true);
    },
    /*
     * Utility Methods
     *
     */
    debounce(func, wait, immediate) {
      var timeout;
      return function () {
        var context = this;
        var args = arguments;
        var later = function () {
          timeout = null;
          if (!immediate) func.apply(context, args);
        };
        var callNow = immediate && !timeout;
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
        if (callNow) func.apply(context, args);
      };
    },
  },
  mounted() {
    // this.show();
     this.initCanvasTools();
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.image-canvas-wrapper {
    width: 100%;
    height: 80vh;
    margin: 0 auto;
}

.image-canvas {
    width: 100%;
    height: 100%;
}
</style>
