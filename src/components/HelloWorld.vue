<template>
  <div>123</div>
</template>

<script>
// 从v2.11.2起，SDK 支持了 WebSocket，推荐接入；v2.10.2及以下版本，使用 HTTP
import TIM from "tim-js-sdk";
import TIMUploadPlugin from "tim-upload-plugin";
// import genTestUserSig from "./GenerateTestUserSig.js";
import "./debug/lib-generate-test-usersig.min.js";
import genTestUserSig from "./debug/gen-test-user-sig.js";

const SDKAPPID = "1400564226";
const EXPIRETIME = 604800;
const SECRETKEY = "e159ef7024ef5e5d7e64e7492bc2c6bf4b02eda2909c42ec94002b8ce33f25a2";

export default {
  name: "HelloWorld",
  data() {
    return {
      msg: "Welcome to Your Vue.js App",
    };
  },
  mounted() {
    console.log("start");
    let options = {
      SDKAppID: 1400564226, // 接入时需要将0替换为您的即时通信 IM 应用的 SDKAppID
    };
    // 创建 SDK 实例，`TIM.create()`方法对于同一个 `SDKAppID` 只会返回同一份实例
    let tim = TIM.create(options); // SDK 实例通常用 tim 表示
    console.log('111111111111', tim);
    // 设置 SDK 日志输出级别，详细分级请参见 <a href="https://web.sdk.qcloud.com/im/doc/zh-cn//SDK.html#setLogLevel">setLogLevel 接口的说明</a>
    tim.setLogLevel(0); // 普通级别，日志量较多，接入时建议使用
    // tim.setLogLevel(1); // release 级别，SDK 输出关键信息，生产环境时建议使用
    // 注册腾讯云即时通信 IM 上传插件
    tim.registerPlugin({ "tim-upload-plugin": TIMUploadPlugin });
    let data = genTestUserSig("ethan");
    console.log('userSig:', data.userSig);
    // 开始登录
    tim.login({
      userID: "ethan",
      userSig: data.userSig
    });
    tim.on(TIM.EVENT.MESSAGE_RECEIVED, function (event) {
      // 收到推送的单聊、群聊、群提示、群系统通知的新消息，可通过遍历 event.data 获取消息列表数据并渲染到页面
      // event.name - TIM.EVENT.MESSAGE_RECEIVED
      // event.data - 存储 Message 对象的数组 - [Message]
      console.error("MESSAGE_RECEIVED:", event);
    });
    // this.sendMessage(tim, "ttttttttttttttttttest");
  },
  methods: {
    sendMessage(tim, text) {
      // 发送文本消息，Web 端与小程序端相同
      // 1. 创建消息实例，接口返回的实例可以上屏
      let message = tim.createTextMessage({
        to: "ethan",
        conversationType: TIM.TYPES.CONV_C2C,
        // 消息优先级，用于群聊（v2.4.2起支持）。如果某个群的消息超过了频率限制，后台会优先下发高优先级的消息，详细请参考：https://cloud.tencent.com/document/product/269/3663#.E6.B6.88.E6.81.AF.E4.BC.98.E5.85.88.E7.BA.A7.E4.B8.8E.E9.A2.91.E7.8E.87.E6.8E.A7.E5.88.B6)
        // 支持的枚举值：TIM.TYPES.MSG_PRIORITY_HIGH, TIM.TYPES.MSG_PRIORITY_NORMAL（默认）, TIM.TYPES.MSG_PRIORITY_LOW, TIM.TYPES.MSG_PRIORITY_LOWEST
        // priority: TIM.TYPES.MSG_PRIORITY_NORMAL,
        payload: {
          text,
        },
        // 消息自定义数据（云端保存，会发送到对端，程序卸载重装后还能拉取到，v2.10.2起支持）
        // cloudCustomData: 'your cloud custom data'
      });
      // 2. 发送消息
      let promise = tim.sendMessage(message);
      promise
        .then(function (imResponse) {
          // 发送成功
          console.log(imResponse);
        })
        .catch(function (imError) {
          // 发送失败
          console.warn("sendMessage error:", imError);
        });
    },
    genTestUserSig(userID) {
      var generator = new window.LibGenerateTestUserSig(
        SDKAPPID,
        SECRETKEY,
        EXPIRETIME
      );
      console.log("generator:", generator);
      var userSig = generator.genTestUserSig(userID);
      console.log("userSig:", userSig);
      return {
        SDKAppID: SDKAPPID,
        userSig: userSig,
      };
    },
  },
};
</script>
<style scoped>
</style>
