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
      <div class="row">
        <div class="col-md-6">
          <!-- Chatbot Section -->
          <div class="chat-section border p-3">
            <h4>Chatbot</h4>
            <textarea
              v-model="chatInput"
              placeholder="Type your message here..."
              class="form-control mb-2"
            ></textarea>
            <button @click="sendMessage" class="btn btn-primary">Send</button>
            <div class="chat-output mt-3">
              <p v-for="(msg, index) in chatMessages" :key="index">{{ msg }}</p>
            </div>
          </div>
        </div>
        <div class="col-md-6">
          <!-- File Summary Section -->
          <div class="file-summary border p-3">
            <h4>File Summary</h4>
            <p>No files uploaded.</p>
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
          </div>
        </div>
        <div class="col-md-6">
          <!-- Output Log Section -->
          <div class="output-log border p-3">
            <h4>Output Log</h4>
            <div v-if="logs">
              <p v-for="(logLine, index) in logs" :key="index">
                <div v-if='!logLine.split(":") || !logLine.split(":")[1]' style="color: black;"> {{ logLine }}</div>
                <div v-if='logLine.split(":") && logLine.split(":")[1]' :style="logLine.split(':')[0] === 'ERROR' ? 'color: red;' : (logLine.split(':')[0] === 'SUCCESS' ? 'color: green;' : 'color: blue;') ">{{ logLine }}</div>
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
    };
  },
  methods: {
    sendMessage() {
      if (this.chatInput.trim()) {
        this.chatMessages.push(this.chatInput);
        this.chatInput = "";
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

p {
  margin: 0;
}
</style>
