<template>
  <!--增加video标签支持，并循环添加-->
  <view :class="node.classStr" :style="node.styleStr" style="position: relative;">
    <div class="video-img-wrapper" @click="play">
      <div class="cover-play">
        <svg viewBox="0 0 64 64" class="Icon Icon--play" width="50" height="50" aria-hidden="true" style="height: 50px; width: 50px;">
          <title></title>
          <g>
            <path fill-opacity="0.9" fill="#fff" d="M32,64 C14.326888,64 0,49.673112 0,32 C0,14.326888 14.326888,0 32,0 C49.673112,0 64,14.326888 64,32 C64,49.673112 49.673112,64 32,64 Z M32.2363929,61.6 C48.5840215,61.6 61.8363929,48.3476286 61.8363929,32 C61.8363929,15.6523714 48.5840215,2.4 32.2363929,2.4 C15.8887643,2.4 2.63639293,15.6523714 2.63639293,32 C2.63639293,48.3476286 15.8887643,61.6 32.2363929,61.6 Z"></path>
            <circle fill-opacity="0.3" fill="#000" cx="32" cy="32" r="29.6"></circle>
            <path fill-opacity="0.9" fill="#fff" d="M43.5634409,30.7271505 C44.6882014,31.4301259 44.6868607,32.5707121 43.5634409,33.2728495 L28.4365591,42.7271505 C27.3117986,43.4301259 26.4,42.9221451 26.4,41.5999847 L26.4,22.4000153 C26.4,21.0745235 27.3131393,20.5707121 28.4365591,21.2728495 L43.5634409,30.7271505 Z"></path>
          </g>
        </svg>
      </div>
      <image :mode="node.attr.mode" :lazy-load="node.attr.lazyLoad" :style="newStyleStr || videoImgStyle" class="video-img" :src="frameSrc" :data-src="frameSrc" @load="wxParseImgLoad" />
    </div>
    <video :id="videoId" :style="videoStyle" @fullscreenchange="fullscreenchange" :class="node.classStr" class="video-video" :src="node.attr.src"></video>
  </view>
</template>

<script>
/**
 * 增加图片节点，展示视频第一帧
 *  -> 视频默认隐藏，展示图片
 *      -> 点击图片，全屏播放视频
 *           -> 退出全屏，视频隐藏
 */
export default {
  name: "wxParseVideo",
  props: {
    node: {}
  },
  data() {
    return {
      newStyleStr: "",
      styleObject: "width: 1px;height:1px;position: absolute;left: 0;top: 0;opacity: 0;z-index: -1"
    };
  },
  computed: {
    videoImgStyle() {
      let videoImgStyle = this.node.attr.videoImgStyle;
      return videoImgStyle ? videoImgStyle : "";
    },
    videoStyle() {
      let videoStyle = this.node.attr.videoStyle;
      return videoStyle ? videoStyle : this.styleObject;
    },
    videoId() {
      return "video_" + new Date().getTime().toString(36);
    },
    frameSrc() {
      let node = this.node;
      let frameUrl = node.attr.frameImgUrl;
      if (node.attr.customFrameImg && frameUrl) {
        return node.replace ? frameUrl : node.attr.src + frameUrl;
      }
      // TODO: JS截取视频帧作为url
      return "";
    }
  },
  methods: {
    fullscreenchange(e) {
      if (!e.target.fullScreen) {
        // 如果退出全屏，则暂停视频
        this.videoContext.pause();
      }
    },
    play() {
      this.videoContext = wx.createVideoContext(this.videoId);
      this.videoContext.play();
      // 全屏
      this.videoContext.requestFullScreen();
    },
    // 图片视觉宽高计算函数区
    wxParseImgLoad(e) {
      // 是否需要全屏展示，如果需要，则根据宽高比，来计算
      const { isFull } = this.node.attr;
      if (!isFull) {
        return;
      }
      const { src } = e.target.dataset;
      if (!src) return;
      const { width, height } = e.mp.detail;
      const recal = this.wxAutoImageCal(width, height);
      const { imageheight, imageWidth } = recal;
      const { mode, videoImgStyle } = this.node.attr;
      const imageHeightStyle = mode === "widthFix" ? "" : `height: ${imageheight}px;`;
      this.newStyleStr = `${videoImgStyle}; ${imageHeightStyle}; width: ${imageWidth}px;`;
    },
    // 计算视觉优先的图片宽高
    wxAutoImageCal(originalWidth, originalHeight) {
      // 获取图片的原始长宽
      const { margin } = this.node.attr;
      const windowWidth = this.node.$screen.width - 2 * margin;
      const results = {};

      if (originalWidth < 60 || originalHeight < 60) {
        const { src } = this.node.attr;
        this.node.$host.removeImageUrl(src);
        this.preview = false;
      }

      // 判断按照那种方式进行缩放
      if (originalWidth > windowWidth) {
        // 在图片width大于手机屏幕width时候
        results.imageWidth = windowWidth;
        results.imageheight = windowWidth * (originalHeight / originalWidth);
      } else {
        // 否则展示原来的数据
        results.imageWidth = originalWidth;
        results.imageheight = originalHeight;
      }

      return results;
    }
  },
  onUnload() {
    if (this.videoContext) {
      this.videoContext = null;
    }
  }
};
</script>

<style lang="scss" scoped>
.video-img-wrapper {
  .cover-play {
    width: 50px;
    height: 50px;
    position: absolute;
    z-index: 2;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);
    background: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 64 64' class='Icon Icon--play' width='50' height='50' aria-hidden='true' style='height: 50px; width: 50px;'%3E %3Ctitle%3E%3C/title%3E %3Cg%3E %3Cpath fill-opacity='0.9' fill='%23fff' d='M32,64 C14.326888,64 0,49.673112 0,32 C0,14.326888 14.326888,0 32,0 C49.673112,0 64,14.326888 64,32 C64,49.673112 49.673112,64 32,64 Z M32.2363929,61.6 C48.5840215,61.6 61.8363929,48.3476286 61.8363929,32 C61.8363929,15.6523714 48.5840215,2.4 32.2363929,2.4 C15.8887643,2.4 2.63639293,15.6523714 2.63639293,32 C2.63639293,48.3476286 15.8887643,61.6 32.2363929,61.6 Z'%3E%3C/path%3E %3Ccircle fill-opacity='0.3' fill='%23000' cx='32' cy='32' r='29.6'%3E%3C/circle%3E %3Cpath fill-opacity='0.9' fill='%23fff' d='M43.5634409,30.7271505 C44.6882014,31.4301259 44.6868607,32.5707121 43.5634409,33.2728495 L28.4365591,42.7271505 C27.3117986,43.4301259 26.4,42.9221451 26.4,41.5999847 L26.4,22.4000153 C26.4,21.0745235 27.3131393,20.5707121 28.4365591,21.2728495 L43.5634409,30.7271505 Z'%3E%3C/path%3E %3C/g%3E %3C/svg%3E");
    background-size: contain;
  }
}
</style>


