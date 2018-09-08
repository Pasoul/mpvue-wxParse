<!--**
 * author: F-loat <chaimaoyuan@foxmail.com>
 *
 * github地址: https://github.com/F-loat/mpvue-wxParse
 *
 * for: Mpvue框架下 微信小程序富文本解析
 */-->

<template>
  <!--基础元素-->
  <div class="wxParse" :class="className" v-if="!loading">
    <block v-for="node of nodes" :key="node.index">
      <wxParseTemplate :node="node" />
    </block>
  </div>
</template>

<script>
import HtmlToJson from "./libs/html2json";
import wxParseTemplate from "./components/wxParseTemplate0";

export default {
  name: "wxParse",
  props: {
    loading: {
      type: Boolean,
      default: false
    },
    className: {
      type: String,
      default: ""
    },
    content: {
      type: String,
      default: ""
    },
    noData: {
      type: String,
      default: '<div style="color: red;">数据不能为空</div>'
    },
    startHandler: {
      type: Function,
      default() {
        return node => {
          node.attr.class = null;
          node.attr.style = null;
        };
      }
    },
    endHandler: {
      type: Function,
      default: null
    },
    charsHandler: {
      type: Function,
      default: null
    },
    imageProp: {
      type: Object,
      default() {
        return {
          mode: "aspectFit",
          padding: 0,
          lazyLoad: false,
          domain: "",
          autoPreview: false,
          styleStr: '' // 图片样式
        };
      }
    },
    videoProp: {
      type: Object,
      default() {
        return {
          isFull: false, // 视频帧图片是否全屏展示
          mode: "aspectFit",
          margin: 0, // 图片边距
          lazyLoad: false, // 视频帧图片是否需要懒加载
          customFrameImg: false, // 自定义视频帧url
          frameImgUrl: "", // 视频帧图片url
          replace: false, // 是否替换视频url
          videoStyle: "", // 自定义video标签样式
          videoImgStyle: "" // 自定义视频帧图片样式
        };
      }
    }
  },
  components: {
    wxParseTemplate
  },
  data() {
    return {
      imageUrls: []
    };
  },
  computed: {
    nodes() {
      const { content, noData, imageProp, videoProp, startHandler, endHandler, charsHandler } = this;
      const parseData = content || noData;
      const customHandler = {
        start: startHandler,
        end: endHandler,
        chars: charsHandler
      };
      const results = HtmlToJson(parseData, customHandler, imageProp, this, videoProp);
      this.imageUrls = results.imageUrls;
      return results.nodes;
    }
  },
  methods: {
    navigate(href, $event) {
      this.$emit("navigate", href, $event);
    },
    preview(src, $event) {
      if (!this.imageUrls.length || !this.imageProp.autoPreview) return;
      wx.previewImage({
        current: src,
        urls: this.imageUrls
      });
      this.$emit("preview", src, $event);
    },
    removeImageUrl(src) {
      const { imageUrls } = this;
      imageUrls.splice(imageUrls.indexOf(src), 1);
    }
  }
};
</script>
