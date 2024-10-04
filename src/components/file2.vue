<template>
  <div class="container mt-4">
    <ul class="nav nav-tabs">
      <li class="nav-item">
        <a
          class="nav-link"
          :class="{ active: activeTab === 'chat' }"
          @click="activeTab = 'chat'"
          href="#"
        >
          Chat
        </a>
      </li>
      <li class="nav-item">
        <a
          class="nav-link"
          :class="{ active: activeTab === 'document' }"
          @click="activeTab = 'document'"
          href="#"
        >
          Document Upload
        </a>
      </li>
    </ul>

    <div v-if="activeTab === 'chat'" class="tab-content mt-4">
      <div class="row" style="width: 60vw; height: 80vh;">
        <div class="col-md-6">
          <!-- Chatbot Section -->
          <div class="main-content-body">
            <div class="main-chat-header">
              <div class="main-img-user">
                <img alt="" src="/src/assets/img/faces/bot2.avif" />
              </div>
              <div class="main-chat-msg-name">
                <h6>Mombasa County Robo</h6>
                <small>Online</small>
              </div>
            </div>
            <!-- main-chat-header -->
            <div
              class="main-chat-body overflow-auto"
              id="ChatBody"
              ref="chatBody"
            >
              <div class="content-inner">
                <div
                  v-for="(message, index) in messages"
                  :key="index"
                  class="media"
                  :class="{
                    'flex-row-reverse': message.sender === 'user',
                  }"
                >
                  <div
                    class="main-img-user"
                    :class="{ online: message.sender === 'user' }"
                  >
                    <img
                      :src="
                        message.sender === 'user'
                          ? '../src/assets/img/faces/9.jpg'
                          : '../src/assets/img/faces/bot.avif'
                      "
                      alt=""
                    />
                  </div>
                  <div class="media-body">
                    <div
                      class="main-msg-wrapper"
                      :class="{
                        right: message.sender === 'user',
                        left: message.sender !== 'user',
                      }"
                    >
                      <span
                        v-if="message.sender === 'bot'"
                        v-html="message.text"
                      ></span>
                      <span v-else>{{ message.text }}</span>
                    </div>
                    <div>
                      <span>{{
                        new Date().toLocaleTimeString([], {
                          hour: '2-digit',
                          minute: '2-digit',
                        })
                      }}</span>
                      <a href="#"
                        ><i class="icon ion-android-more-horizontal"></i
                      ></a>
                    </div>
                  </div>
                </div>
              </div>
            </div>
            <!-- Main chat footer -->
            <div class="main-chat-footer" style="width: 30vw ; ">
              <input
                class="form-control"
                v-model="userMessage"
                placeholder="Type your message here..."
                type="text"
                @keyup.enter="sendMessage"
              />
              <a class="main-msg-send" @click="sendMessage">
                <i class="far fa-paper-plane"></i>
              </a>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <!-- File Summary Section -->
          <div class="file-summary border p-3">
            <h4>File Summary</h4>
            <div v-if="!fileSummary || fileSummary.length === 0">No file summary</div>
            <div v-if="fileSummary" v-for="summary in fileSummary">
              <p>{{ summary }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>

    <div
      v-if="activeTab === 'document'"
      class="tab-content mt-4"
      style="width: 60vw"
    >
      <div class="row">
        <div class="col-md-6">
          <!-- Document Upload Section -->
          <div class="upload-section border p-3">
            <h4>Upload Document</h4>
            <input
              type="file"
              @change="handleFileUpload"
              class="form-control mb-2"
            />
            <button @click="uploadDocument" class="btn btn-primary">
              Upload
            </button>
            <div v-if="loading" class="loading-spinner">
              <div class="spinner"></div>
              <p>Loading...</p>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <!-- Output Log Section -->
          <div class="output-log border p-3">
            <h4>Output Log</h4>
            <div v-if="logs">
              <p v-for="(logLine, index) in logs" :key="index">
                <div
                  v-if="!logLine.split(':') || !logLine.split(':')[1]"
                  style="color: black;"
                >
                  {{ logLine }}
                </div>
                <div
                  v-if="logLine.split(':') && logLine.split(':')[1]"
                  :style="
                    logLine.split(':')[0] === 'ERROR'
                      ? 'color: red;'
                      : logLine.split(':')[0] === 'SUCCESS'
                      ? 'color: green;'
                      : 'color: blue;'
                  "
                >
                  {{ logLine }}
                </div>
              </p>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      activeTab: "chat", // Default tab
      chatInput: "",
      chatMessages: [],
      uploadLog: "",
      selectedFile: null, // Store the selected file
      logs: [],
      loading: false, // Loading state
      userMessage: "", // User input message
      messages: [], // Array to hold chat messages
      fileSummary: []
    };
  },
  methods: {
    async sendMessage() {
      const message = this.userMessage.trim(); // Trim whitespace
      if (message) {
        // Add user message to the chat
        this.messages.push({ sender: "user", text: message });
        this.userMessage = ""; // Clear input field after sending message
        // upload message
        try {
            const response = await axios.post(
              "http://192.168.0.195:8000/api/query",
              {
                query: message
              }
            );
            const chunks = response.data.chunks[0];
            this.fileSummary.push(
              response.data.context,
              `chunk id: ${response.data.chunks[0].chunk_id}`,
              `doc name: ${response.data.chunks[0].doc_name}`,
              `doc uuid: ${response.data.chunks[0].doc_uuid}`,
              `score: ${response.data.chunks[0].score}`,
              `text: ${response.data.chunks[0].text}`
            );
            console.log("Response received:", response.data);
          } catch (error) {
            // Handle any errors
            console.error("Error:", error);
            alert("Failed to fetch response.");
          }
      }
    },
    handleFileUpload(event) {
      this.selectedFile = event.target.files[0]; // Get the selected file
      this.uploadLog = this.selectedFile.name; // Store the file name
    },
    // Function to convert ArrayBuffer to Base64
    arrayBufferToBase64(buffer) {
      let binary = "";
      const bytes = new Uint8Array(buffer);
      const len = bytes.byteLength;
      for (let i = 0; i < len; i++) {
        binary += String.fromCharCode(bytes[i]);
      }
      return window.btoa(binary);
    },
    async uploadDocument() {
      if (this.uploadLog) {
        const fileExtension = this.uploadLog.split(".").pop(); // Extract the file extension
        const filename = this.uploadLog.split(".")[0]; // Get the filename without extension

        this.loading = true; // Set loading state to true

        // Use FileReader to read the file as an ArrayBuffer
        const fileReader = new FileReader();
        fileReader.onload = async (e) => {
          const arrayBuffer = e.target.result;
          const base64Content = this.arrayBufferToBase64(arrayBuffer);

          try {
            const response = await axios.post(
              "http://192.168.0.195:8000/api/import",
              {
                data: [
                  {
                    filename: filename,
                    extension: fileExtension,
                    content: base64Content, // Send the Base64-encoded file content
                  },
                ],
                textValues: [""],
                config: {},
              }
            );

            // Handle the successful response
            this.logs = [];
            this.logs.push("File uploaded successfully!");
            response.data.logging.forEach((element) => {
              this.logs.push(element.type + ": " + element.message);
            });
            console.log("File uploaded successfully:", response.data);
          } catch (error) {
            // Handle any errors
            console.error("Error uploading file:", error);
            this.logs = "Failed to upload the file.";
            alert("Failed to upload the file.");
          } finally {
            this.loading = false; // Set loading state to false
          }
        };

        // Read the file as an ArrayBuffer (suitable for binary and text)
        fileReader.readAsArrayBuffer(this.selectedFile);
      } else {
        alert("Please select a file before uploading.");
      }
    },
  },
};
</script>

<style scoped>
.nav-tabs .nav-link {
  cursor: pointer;
  background: rgb(237, 237, 237,1);
  backdrop-filter: blur(10px);
}

.chat-section,
.file-summary,
.upload-section,
.output-log {
  height: 300px;
  overflow-y: auto;
}

textarea {
  resize: none;
  height: 100px;
}

.border {
  border: 1px solid #ddd;
}

.loading-spinner {
  display: flex;
  align-items: center;
}

.spinner {
  border: 4px solid rgba(0, 0, 0, 0.1);
  border-left-color: #3498db;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  animation: spin 1s linear infinite;
  margin-right: 10px;
}

@keyframes spin {
  to {
    transform: rotate(360deg);
  }
}

/* Add the following CSS for the fixed input area */
.main-chat-footer {
  position: sticky;
  bottom: 0;
  padding: 10px;
  background-color: #fff;
  border-top: 1px solid #ddd;
  width: 100%; /* Full width within its container */
}

.main-chat-body {
  height: calc(80vh - 70px); /* Adjust for footer height */
  overflow-y: auto;
  padding-bottom: 60px; /* Space for footer */
}

</style>
