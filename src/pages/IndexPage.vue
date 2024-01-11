<template>
  <q-page class="flex flex-center">
    <div>
      <q-uploader
        style="max-width: 300px"
        url="http://localhost:5000/process"
        label="Upload .pdf/.txt file"
        accept=".pdf, .txt"
        auto-upload
        max-files="1"
        @uploaded="
          (info) => {
            console.log(info.xhr);
            textChunks = JSON.parse(info.xhr.response).chunks;
            console.log(textChunks);
          }
        "
      />
      <q-input square outlined v-model="userText" label="Query" />
      <q-btn
        :loading="loading"
        color="secondary"
        @click="askQuery(userText)"
        label="Ask"
      />
    </div>
  </q-page>
</template>

<script setup>
import { ref } from "vue";
import axios from "axios";

let loading = ref(false);
let userText = ref("");
let textChunks = ref([]);

async function askQuery(query) {
  loading.value = true;
  let payload = {
    query: query,
    chunks: textChunks.value,
  };
  axios.post("http://localhost:5000/answer", payload).then((response) => {
    console.log(response);
    loading.value = false;
  });
}
</script>
