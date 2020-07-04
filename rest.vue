<template>
  <section class="grid">
    <div class="controls">
      <h1>Rest (axios / jwt)</h1>
      <div class="actions">
        <button @click="GET">GET</button>
        <button @click="POST">POST</button>
        <button @click="PUT">PUT</button>
        <button @click="DELETE">DELETE</button>
      </div>
      <div class="form">
        <input type="text" v-model="url" placeholder="url" />
        <input type="text" v-model="token" placeholder="jwt token" class="token" />
        <div>
          <input type="text" v-model="fileID" placeholder="file id" />
          <input type="file" @change="previewFile" />
        </div>
        <textarea rows="10" v-model="body" placeholder="key: value"></textarea>
        <pre>{{ jsonPreview ? jsonPreview : 'JSON Mal Formatado' }}</pre>
      </div>
    </div>
    <div class="response">
      <transition mode="out-in">
        <div class="result" v-if="data">
          <h3>Status</h3>
          <pre v-if="status" v-html="status"></pre>
          <h3>Data</h3>
          <pre v-if="data" v-html="data"></pre>
          <h3>Headers</h3>
          <pre v-if="headers" v-html="headers"></pre>
          <h3>Request</h3>
          <pre v-if="request" v-html="request"></pre>
          <h3>Config</h3>
          <pre v-if="config" v-html="config"></pre>
        </div>
      </transition>
    </div>
    <footer>🐺</footer>
  </section>
</template>

<script>
/* eslint-disable */
import axios from "axios";

function formatJSON(json) {
  json = JSON.stringify(json, null, 1);
  json = json.replace(/["][-\w]+["]\:/g, "<span>$&</span>");
  return json;
}

export default {
  data() {
    return {
      url: "",
      body: "",
      token: "",
      fileID: "",
      file: "",
      data: null,
      headers: null,
      request: null,
      status: null,
      config: null
    };
  },
  computed: {
    jsonPreview() {
      try {
        return JSON.parse(this.body);
      } catch (error) {
        console.error("Erro na chave", error);
      }
    },
    jsonBody() {
      if (this.body) {
        try {
          const formData = new FormData();
          const object = JSON.parse(this.body);
          const keys = Object.keys(object);
          keys.forEach(key => {
            formData.append(key, object[key]);
          });
          if (this.file && this.fileID) {
            formData.append(this.fileID, this.file);
          }
          return formData;
        } catch (error) {
          console.error("Erro na chave", error);
        }
      }
    }
  },
  methods: {
    previewFile(event) {
      this.file = event.target.files[0];
    },
    resetData() {
      this.data = null;
      this.headers = null;
      this.request = null;
      this.status = null;
      this.config = null;
    },
    formatResponse(response) {
      console.log(response);
      this.data = formatJSON(response.data);
      this.headers = formatJSON(response.headers);
      this.status = formatJSON(response.status);
      this.config = formatJSON(response.config);
      return response;
    },
    formatError(error) {
      console.log(error);
      this.data = formatJSON(error);
      this.status = error.response.status;
      return error;
    },
    GET() {
      this.resetData();
      this.jwt();
      axios
        .get(this.url)
        .then(this.formatResponse)
        .catch(this.formatError);
    },
    POST() {
      this.resetData();
      this.jwt();
      axios
        .post(this.url, this.jsonBody)
        .then(this.formatResponse)
        .catch(this.formatError);
    },
    PUT() {
      this.resetData();
      this.jwt();
      axios
        .put(this.url, this.jsonBody)
        .then(this.formatResponse)
        .catch(this.formatError);
    },
    DELETE() {
      this.resetData();
      this.jwt();
      axios
        .delete(this.url, this.jsonBody)
        .then(this.formatResponse)
        .catch(this.formatError);
    },
    jwt() {
      if (this.token) {
        axios.defaults.headers.common["Authorization"] = "Bearer " + this.token;
      }
    }
  },
  watch: {
    url() {
      window.localStorage.url = this.url;
    },
    body() {
      window.localStorage.body = this.body;
    },
    fileID() {
      window.localStorage.fileID = this.fileID;
    },
    token() {
      if (!this.token) {
        delete axios.defaults.headers.common["Authorization"];
      }
      window.localStorage.token = this.token;
    }
  },
  created() {
    this.url =
      window.localStorage.url !== "undefined" ? window.localStorage.url : "";
    this.body =
      window.localStorage.body !== "undefined" ? window.localStorage.body : "";
    this.fileID =
      window.localStorage.fileID !== "undefined"
        ? window.localStorage.fileID
        : "";
    this.token =
      window.localStorage.token !== "undefined"
        ? window.localStorage.token
        : "";
  }
};
</script>

<style>
body {
  background: #222;
  font-family: "Avenir", Arial, Helvetica, sans-serif;
  padding: 20px;
  margin: 0px;
}

.grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  grid-gap: 30px;
}

h1 {
  color: #fff;
  margin-top: 0px;
}

h3 {
  font-weight: normal;
  font-family: "IBM Plex Mono", monospace;
  color: white;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.1rem;
}

pre {
  font-size: 1rem;
  color: rgba(255, 255, 255, 0.7);
  font-family: "IBM Plex Mono", monospace;
}
span {
  color: greenyellow;
}

.form {
  display: grid;
  grid-gap: 10px;
}

input,
textarea {
  font-size: 0.9rem;
  font-family: "IBM Plex Mono", monospace;
  border: none;
  padding: 10px;
  border-radius: 2px;
}

input[type="file"] {
  color: white;
}

textarea {
  font-size: 1rem;
}

.actions {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  grid-gap: 10px;
  margin-top: 20px;
  margin-bottom: 10px;
}

.token {
  font-size: 0.8rem;
}

button {
  font-size: 1rem;
  border: none;
  border-radius: 2px;
  padding: 10px 0;
  background: #87f;
  color: white;
  cursor: pointer;
}

.result {
  overflow-x: scroll;
}

.result::-webkit-scrollbar {
  width: 10px;
  height: 5px;
}

.result::-webkit-scrollbar-track {
  background: #222;
  border-radius: 0px;
}

.result::-webkit-scrollbar-thumb {
  background: greenyellow;
  border-radius: 0px;
}

.v-enter {
  opacity: 0;
  transform: translate3d(0, -20px, 0);
}

.v-leave-to {
  opacity: 0;
  transform: translate3d(0, 20px, 0);
}

.v-enter-active,
.v-leave-active {
  transition: all 0.3s;
}

@media screen and (min-width: 700px) {
  .controls {
    min-height: 100vh;
  }
}

footer {
  grid-row: 2;
}
</style>
