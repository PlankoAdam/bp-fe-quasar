<template>
  <q-page class="flex flex-center">
    <div
      class="min-w-fit sm:max-w-prose w-full m-5 flex flex-col content-center"
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
            textChunks = JSON.parse(info.xhr.response).chunks;
          }
        "
      />
      <div class="flex flex-row w-full my-3">
        <q-input
          class="flex-1 me-3"
          square
          outlined
          v-model="userText"
          label="Query"
        />
        <q-btn
          :loading="loading"
          color="secondary"
          @click="askQuery(userText)"
          label="Ask"
        />
      </div>
      <div class="text-justify text-wrap max-w-prose w-full">
        <h1 class="text-xl mt-5">Answer:</h1>
        <p class="min-h-20 px-3">{{ answer }}</p>
        <h1 class="text-xl mt-5">Context:</h1>
        <p class="min-h-20 px-3 italic">"...{{ ctx }}..."</p>
      </div>
    </div>
  </q-page>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

let loading = ref(false);
let userText = ref("");
let textChunks = ref([]);

let answer = ref("");
let ctx = ref("");

async function askQuery(query) {
  loading.value = true;
  let payload = {
    query: query,
    chunks: textChunks.value,
  };
  axios
    .post("http://localhost:5000/answer", payload)
    .then((response) => {
      console.log(response.data.answer);
      console.log(response.data.context);
      answer.value = response.data.answer;
      ctx.value = response.data.context;
    })
    .finally(() => {
      loading.value = false;
    });
}
</script>
