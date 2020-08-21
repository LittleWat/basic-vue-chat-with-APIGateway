<template>
  <div>
    <basicvuechat
      :title="'Sample Chat Vue App'"
      :initial-feed="messages"
      @newOwnMessage="onNewOwnMessage"
    />
  </div>
</template>

<script>
import Vue from "vue";
import basicvuechat from "basic-vue-chat";
Vue.use(basicvuechat);

// チャット履歴をためる変数
var feed = [];

let connection = new WebSocket(
  "wss://{YOUR-API-ID}.execute-api.{YOUR-REGION}.amazonaws.com/Prod"
);

// 接続した際のコールバック
connection.onopen = function() {
  // メッセージの送信
  // connection.send(`こんにちは世界`);
};

// エラーが出た場合
connection.onerror = function(error) {
  console.error(error);
};

// メッセージを受け取った場合
connection.onmessage = function(event) {
  const result = JSON.parse(event.data);
  console.log(`onmessage: ${JSON.stringify(event)}`);

  if (result.senderConnectionId == result.receiverConnectionId) {
    console.log("自分の送ったものが届いたよ");
    return;
  }

  var now = new Date();
  feed.push({
    author: result.senderConnectionId, // FIXME: result.name に置き換える
    imageUrl: "",
    contents: result.message,
    date: `${now.getHours()}:${now.getMinutes()}:${now.getSeconds()}`,
  });
  console.log(`feed: ${feed}`);
};

export default {
  components: {
    basicvuechat,
  },
  data() {
    return {
      messages: feed, 
    };
  },
  methods: {
    onNewOwnMessage(message) {
      console.log(`message: ${message}, messageType: ${typeof message} `);
      connection.send(
        JSON.stringify({
          action: "sendmessage",
          data: {
            message: message,
            name: "マイケル", // FIXME: cogitoなどから名前を取得し設定する
          },
        })
      );
    },
  },
  created() {},
};
</script>
