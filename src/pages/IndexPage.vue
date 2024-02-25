<template>
  <q-page class="flex flex-col content-center w-full h-full min-h-full">
    <div
      v-if="!uploaded"
      class="min-w-fit sm:max-w-xl max-w-2xl w-full flex flex-col min-h-full h-full flex-1 justify-evenly px-8"
    >
      <q-uploader
        class="w-full"
        url="http://localhost:5000/process"
        label="Upload .pdf/.txt file"
        accept=".pdf, .txt"
        auto-upload
        max-files="1"
        @uploaded="
          (info) => {
            uploaded = true;
            textChunks = JSON.parse(info.xhr.response).chunks;
          }
        "
      />
    </div>

    <div
      v-if="uploaded"
      class="sm:max-w-3xl max-w-2xl w-full flex flex-col min-h-full h-full flex-1"
    >
      <div class="q-pa-md flex flex-col justify-end mb-20">
        <div ref="chatDiv" class="w-full max-h-full">
          <q-chat-message
            v-for="(msg, index) in chatHistory"
            :key="index"
            :sent="msg.sent"
            .bg-color="
              () => {
                if (true) {
                  console.log('bgcolor');
                  return 'primary';
                }
              }
            "
          >
            <div v-for="(t, index) in msg.text" :key="index">{{ t }}</div>
            <q-spinner-dots v-if="msg.waiting" />
          </q-chat-message>
        </div>
      </div>
      <div
        class="flex flex-row p-3 fixed bottom-0 bg-white"
        style="width: inherit; max-width: inherit"
      >
        <q-input
          class="flex-1 me-3"
          square
          outlined
          v-model="userText"
          label="Query"
          @keyup.enter="askQuery(userText)"
          :readonly="loading"
        />
        <q-btn
          :loading="loading"
          color="secondary"
          @click="askQuery(userText)"
          label="Ask"
        />
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

let uploaded = ref(false);

let loading = ref(false);
let userText = ref("");
let textChunks = ref([]);

let chatHistory = ref([]);
let chatDiv = ref(null);

let msgs = [];

async function askQuery(query) {
  userText.value = "";

  chatHistory.value.push({
    text: [query],
    sent: true,
    waiting: false,
  });
  chatHistory.value.push({
    text: [],
    sent: false,
    waiting: true,
  });
  scroll();

  loading.value = true;

  let payload = {
    query: query,
    chunks: textChunks.value,
    msgs: msgs,
  };
  axios
    .post("http://localhost:5000/answer", payload)
    .then((response) => {
      msgs.push(query);
      msgs.push(response.data.answer);
      chatHistory.value.pop();
      chatHistory.value.push({
        text: [response.data.context, response.data.answer],
        sent: false,
        waiting: false,
      });
      scroll();
      console.log(response.data.answer);
      console.log(response.data.context);
    })
    .finally(() => {
      loading.value = false;
    });
}

function scroll() {
  chatDiv.value.scrollTop = chatDiv.value.scrollHeight;
}
</script>
