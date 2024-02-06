<template>
  <q-page class="flex flex-col content-center w-full h-full min-h-full">
    <div
      class="min-w-fit sm:max-w-3xl w-full flex flex-col min-h-full h-full flex-1 items-center"
    >
      <q-uploader
        v-if="!uploaded"
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
      <div v-if="uploaded" style="width: inherit; max-width: inherit">
        <div class="q-pa-md flex flex-col justify-end mb-20">
          <div class="w-full max-h-full">
            <q-chat-message
              v-for="(msg, index) in chatHistory"
              :key="index"
              :sent="msg.sent"
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
          />
          <q-btn
            :loading="loading"
            color="secondary"
            @click="askQuery(userText)"
            label="Ask"
          />
        </div>
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

  loading.value = true;

  let payload = {
    query: query,
    chunks: textChunks.value,
  };
  axios
    .post("http://localhost:5000/answer", payload)
    .then((response) => {
      chatHistory.value.pop();
      chatHistory.value.push({
        text: [response.data.context, response.data.answer],
        sent: false,
        waiting: false,
      });
      console.log(response.data.answer);
      console.log(response.data.context);
    })
    .finally(() => {
      loading.value = false;
    });
}
</script>
