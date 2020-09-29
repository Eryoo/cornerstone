<template>
  <div>
    <div class="image-canvas-wrapper" oncontextmenu="return false">
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
      dcmDataList: [
        {
          fileName: "c31ac6c3-8453-4fde-add7-b54bc9e3ca75.dcm",
          fileUrl:
            "http://oss.lrhealth.com/new-ihp/c31ac6c3-8453-4fde-add7-b54bc9e3ca75.dcm",
          fileUuid: "c31ac6c3-8453-4fde-add7-b54bc9e3ca75",
          instanceId: "2723252330215587840",
          seriesId: "2723252330177839104",
          sopInstanceUid:
            "1.3.6.1.4.1.14519.5.2.1.6279.6001.161876394176451257335687660264",
          sortNo: 80,
          studyId: "2723252330018455552",
        },
        {
          fileName: "0fd109c0-de9a-4865-9211-451d1c1869de.dcm",
          fileUrl:
            "http://oss.lrhealth.com/new-ihp/0fd109c0-de9a-4865-9211-451d1c1869de.dcm",
          fileUuid: "0fd109c0-de9a-4865-9211-451d1c1869de",
          instanceId: "2723571195109785600",
          seriesId: "2723252330177839104",
          sopInstanceUid:
            "1.3.6.1.4.1.14519.5.2.1.6279.6001.196168146672553694213150545775",
          sortNo: 59,
          studyId: "2723252330018455552",
        },
        {
          fileName: "413cae79-7b3d-4f3a-b2ce-55e0c5494439.dcm",
          fileUrl:
            "http://oss.lrhealth.com/new-ihp/413cae79-7b3d-4f3a-b2ce-55e0c5494439.dcm",
          fileUuid: "413cae79-7b3d-4f3a-b2ce-55e0c5494439",
          instanceId: "2723571198091935744",
          seriesId: "2723252330177839104",
          sopInstanceUid:
            "1.3.6.1.4.1.14519.5.2.1.6279.6001.119677692049347358854983062958",
          sortNo: 52,
          studyId: "2723252330018455552",
        },
        {
          fileName: "8b95464a-a460-4ff5-a078-2a126f31d9c0.dcm",
          fileUrl:
            "http://oss.lrhealth.com/new-ihp/8b95464a-a460-4ff5-a078-2a126f31d9c0.dcm",
          fileUuid: "8b95464a-a460-4ff5-a078-2a126f31d9c0",
          instanceId: "2723571203536142336",
          seriesId: "2723252330177839104",
          sopInstanceUid:
            "1.3.6.1.4.1.14519.5.2.1.6279.6001.311972233297020504390662508317",
          sortNo: 42,
          studyId: "2723252330018455552",
        },
      ],
    };
  },
  methods: {
    getimg() {
      let self = this;
      axios({
        method: "get",
        url: "/api/000001.dcm",
        responseType: "arrayBuffer",
      }).then((res) => {
        let element = document.getElementById("canvas");
        let imageId = cornerstoneWADOImageLoader.wadouri.fileManager.add(
          res.data
        );
        cornerstone.loadAndCacheImage(imageId).then(
          function (image) {
            var viewport = cornerstone.getDefaultViewportForImage(
              element,
              image
            );
            cornerstone.displayImage(element, image, viewport);
          },
          function (err) {
            console.log(err);
          }
        );
      });
    },
    show(img) {
      // this.getimg();
      // return;
      const _this = this;
      if (this.isShow === true) {
        this.isShow = false;
        axios({
          method: "get",
          url: "/api/000001.dcm",
          responseType: "arrayBuffer",
        }).then(function (res) {
          // 找到要渲染的元素
          // let canvas = _this.$refs.canvas;
          // // 在 DOM 中将 canvas 元素注册到 cornerstone
          // cornerstone.enable(canvas);
          // let imageId =
          //   "wadouri:" +
          //   "http://oss.lrhealth.com/new-ihp/0fd109c0-de9a-4865-9211-451d1c1869de.dcm";
          // const imageIds = [
          //   `wadouri:https://tools.cornerstonejs.org/examples/assets/dicom/bellona/chest_lung/1.dcm`,
          //   `wadouri:http://oss.lrhealth.com/new-ihp/0fd109c0-de9a-4865-9211-451d1c1869de.dcm`,
          //   `wadouri:http://oss.lrhealth.com/new-ihp/413cae79-7b3d-4f3a-b2ce-55e0c5494439.dcm`,
          //   `wadouri:http://oss.lrhealth.com/new-ihp/8b95464a-a460-4ff5-a078-2a126f31d9c0.dcm`,
          // ];

          // http://test.lrhealth.com/000001.dcm
          // cornerstone.loadAndCacheImage(imageIds[0]).then(
          //   function (image) {
          //     console.log(image);
          //     // 设置元素视口
          //     var viewport = cornerstone.getDefaultViewportForImage(
          //       canvas,
          //       image
          //     );
          //     cornerstoneTools.addStackStateManager(canvas, ["stack"]);
          //     cornerstoneTools.addToolState(canvas, "stack", stack);
          //     // 显示图像
          //     cornerstone.displayImage(canvas, image, viewport);
          //     // 激活工具
          //     _this.initCanvasTools();
          //   },
          //   function (err) {
          //     console.log(err);
          //   }
          // );
          // cornerstone.loadImage(imageIds[0]).then((image) => {
          //   var viewport = cornerstone.getDefaultViewportForImage(
          //     canvas,
          //     image
          //   );
          //   cornerstoneTools.addStackStateManager(canvas, ["stack"]);
          //   cornerstoneTools.addToolState(canvas, "stack", stack);
          //   // 显示图像
          //   cornerstone.displayImage(canvas, image, viewport);
          _this.initCanvasTools();
          // });

          // Dicom 加载 进度
          cornerstone.events.addEventListener(
            "cornerstoneimageloadprogress",
            function (event) {
              const eventData = event.detail;
              const loadProgress = document.getElementById("loadProgress");
              loadProgress.textContent = `Dicom加载: ${eventData.percentComplete}%`;
            }
          );
        });
      } else {
        this.isShow = true;
      }
    },
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
    this.show();
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
