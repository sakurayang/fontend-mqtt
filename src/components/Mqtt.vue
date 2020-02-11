<template>
  <div class="mqttbox">
    <div class="publish-header header">发布消息</div>
    <div class="publish">
      <div
        class="message-box"
        :class="message.level"
        v-for="message of publish_data"
        :key="message.id"
      >
        <div class="message-time">
          发送时间：{{
            new Date(message.time).toLocaleString("zh-CN-u-chinese-h24", {
              hourCycle: "h24",
              formatMatcher: "basic"
            })
          }}
        </div>
        <div class="message-sender">发送至：{{ message.to }}</div>
        <div class="message-text" :id="message.id">{{ message.text }}</div>
        <div class="message-status" :class="message.status" />
      </div>
    </div>
    <div class="publish-footer footer">
      <select class="publish-select" v-model="select" placeholder="主题">
        <option
          v-for="topic in topics"
          :key="topic.name"
          :value="topic.name"
          class="select-item"
        >
          {{ topic.name }} - {{ topic.desc }}
        </option>
      </select>
      <input
        type="text"
        class="public-input"
        v-model="message"
        id="public-input"
        @keyup.enter="sendMessage(select, message)"
        autofocus="true"
        placeholder="输入你需要发布的信息"
      />
      <button
        type="button"
        class="public-submit"
        @click="sendMessage(select, message)"
      >
        提交
      </button>
    </div>
    <hr class="vertical" />
    <div class="receive-header header">接收消息</div>
    <div class="receive">
      <div
        class="message-box"
        :class="message.level"
        v-for="message of receive_data"
        :key="message.id"
      >
        <div class="message-time">
          接收时间：{{
            new Date(Date.now()).toLocaleString("zh-CN-u-chinese-h24", {
              hourCycle: "h24",
              formatMatcher: "basic"
            })
          }}
        </div>
        <div class="message-topic">主题：{{ message.to }}</div>
        <div
          class="message-text"
          :id="message.id"
          :title="`发送自${message.from}`"
        >
          {{ message.text }}
        </div>
        <div class="message-status" />
      </div>
    </div>
    <div class="receive-footer footer" />
  </div>
</template>

<script>
import uuid from "uuid";
import mqtt from "async-mqtt";
import config from "@/utils/config";
//import { url } from "url";
export default {
  name: "Mqtt",
  data: function() {
    let topics = config.mqtt.subscribe;
    let publish_data = [];
    let receive_data = [];
    let this_id = uuid.v4();
    async function init() {
      let client = await mqtt.connectAsync(
        // url.parse(`${config.mqtt.host}:${config.mqtt.port}`),
        `${config.mqtt.host}:${config.mqtt.port}${config.mqtt.path}`,
        {
          username: config.mqtt.user,
          password: config.mqtt.password
        }
      );
      for (const topic of config.mqtt.subscribe) {
        //console.log(topic.name);
        client.subscribe(topic.name);
      }
      client.on("error", err => console.error(err));
      //client.on("messgae", (t, m, p) => console.log(t, m, p));
      client.on("packetreceive", parsePacket);
      window.mqtt_client = client;
    }
    init();
    async function parsePacket(packet) {
      let cmd = packet.cmd;
      if (cmd !== "publish") return;
      let data = packet.payload.toString();
      try {
        data = JSON.parse(data);
        let id = data.id;
        if (receive_data.length !== 0) {
          for (const store of receive_data) {
            if (store.id === id) return;
          }
        }
        receive_data.push(data);
      } catch (error) {
        console.log("packet format error", data);
        return;
      }
      //console.log(packet);
      let message_window = document.getElementsByClassName("receive")[0];
      message_window.scrollTo({
        top:
          message_window.lastChild.offsetTop * 2 +
          message_window.lastChild.clientHeight +
          message_window.scrollHeight,
        behavior: "smooth"
      });
    }

    async function sendMessage(topic, message) {
      if (!topic || !message) return alert("未选择topic或消息为空");
      let id = uuid.v4();
      publish_data.push({
        id,
        time: Date.now(),
        from: this_id,
        to: topic,
        text: message,
        status: "sending"
      });
      try {
        await window.mqtt_client.publish(
          topic,
          JSON.stringify({
            id,
            time: Date.now(),
            from: this_id,
            to: topic,
            text: message
          })
        );
        publish_data[publish_data.length - 1].status = "sent";
      } catch (error) {
        console.error(error);
        publish_data[publish_data.length - 1].status = "faild";
      }
      message = "";
      document.getElementById("public-input").value = "";
      let message_window = document.getElementsByClassName("publish")[0];
      message_window.scrollTo({
        top:
          message_window.lastChild.offsetTop +
          message_window.lastChild.clientHeight +
          message_window.scrollHeight,
        behavior: "smooth"
      });
    }

    return {
      publish_data,
      receive_data,
      topics,
      sendMessage,
      parsePacket,
      message: "",
      select: ""
    };
  }
};
</script>

<style scoped>
.mqttbox {
  display: grid;
  grid-template:
    "publish-header receive-header" minmax(5vh, auto)
    "publish receive" 90vh
    "publish-footer receive-footer" minmax(5vh, auto)
    / 50vw 50vw;
}
.vertical {
  position: fixed;
  top: 50%;
  left: 50%;
  width: 103vh;
  border: 3px solid black;
  transform: translate(-50%, -50%) rotate(90deg);
}
.header {
  min-height: max-content;
  font-size: 4vh;
  text-align: center;
  border-bottom: 1.5px solid black;
}
.footer {
  min-height: max-content;
  border-top: 1.5px solid black;
}
.publish-header {
  grid-area: publish-header;
}
.publish-footer {
  grid-area: publish-footer;
  display: grid;
  grid-template: "select text submit" minmax(5vh, 2.5rem) / 1.5fr 8fr 1fr;
}
.receive-header {
  grid-area: receive-header;
}
.receive-footer {
  grid-area: receive-footer;
}

.publish,
.receive {
  overflow-y: auto;
  overflow-wrap: break-word;
  line-break: normal;
}

.message-box {
  display: grid;
  grid-template:
    "time . . status" 0.8rem
    "sender . . ." 0.8rem
    "text text text text" minmax(1.5rem, auto)
    / minmax(auto, 11vw) minmax(auto, 11vw) minmax(auto, 11vw) minmax(auto, 11vw);
  padding-left: 1rem;
  padding-right: 1rem;
  padding-top: 0.5rem;
}
.message-time {
  grid-area: time;
  font-size: 0.7rem;
  color: #6e6e6e;
}
.message-sender,
.message-topic {
  grid-area: sender;
  font-size: 0.7rem;
  color: #6e6e6e;
}
.message-text {
  grid-area: text;
  font-size: 1.2rem;
}
.message-time,
.message-sender,
.message-status,
.message-topic {
  filter: opacity(0.5);
}
.message-time:hover,
.message-sender:hover,
.message-status:hover,
.message-topic:hover {
  filter: opacity(1);
}
.message-time,
.message-time:hover,
.message-sender,
.message-sender:hover,
.message-topic,
.message-topic:hover,
.message-status,
.message-status:hover,
.public-input,
.public-input:hover {
  transition-delay: 0;
  transition-duration: 300ms;
  transition-property: all;
  transition-timing-function: cubic-bezier(0.59, 0.04, 0.1, 1.03);
}
.message-status {
  position: relative;
  right: 0;
  text-align: end;
  grid-area: status;
}
.sending::after {
  color: #e56f08;
  font-size: 0.7rem;
  content: "📧发送中";
}
.faild::after {
  color: #e30000;
  font-size: 0.7rem;
  content: "⚠发送失败";
}
.sent::after {
  color: #009200;
  font-size: 0.7rem;
  content: "✔已发送";
}

.publish-select {
  border: none;
  height: 100%;
}
.public-input {
  width: 100%;
  height: 100%;
  color: black;
  font-size: minmax(4vh, 1.5rem);
  border: none;
  border-bottom: 1.5px solid grey;
}
.public-input:hover {
  border-bottom: 1.5px solid black;
}
.public-submit {
  width: 100%;
  height: 100%;
  background: none;
  box-shadow: none;
  border: none;
}
</style>
