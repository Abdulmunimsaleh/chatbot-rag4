<template>
  <div class="container mt-4" style="width: 1200vw;">
    <ul class="nav nav-tabs">
      <li class="nav-item">
        <a
          class="nav-link"
          :class="{ active: activeTab === 'chat', 'tab-highlight': activeTab === 'chat' }"
          @click="activeTab = 'chat'"
          href="#"
        >
          Chat
        </a>
      </li>
      <li class="nav-item">
        <a
          class="nav-link"
          :class="{ active: activeTab === 'document', 'tab-highlight': activeTab === 'document' }"
          @click="activeTab = 'document'"
          href="#"
        >
          Document Upload
        </a>
      </li>
    </ul>

    <div v-if="activeTab === 'chat'" class="tab-content mt-4" style="width: 100vw;">
      <div class="row" style="width: 100vw; height: 80vh;">
        <div class="col-md-6">
          <!-- Chatbot Section -->
          <div class="main-content-body">
            <div class="main-chat-header">
              <div class="main-img-user">
                <img alt="" src="/src/assets/img/faces/bot2.avif" />
              </div>
              <div class="main-chat-msg-name">
                <h6>Chatbot</h6>
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
                 <!-- Typing Indicator -->
                 <div v-if="botTyping" class="media">
                  <div class="main-img-user">
                    <img src="/src/assets/img/faces/bot.avif" alt="" />
                  </div>
                  <div class="media-body">
                    <div class="main-msg-wrapper">
                      <div class="typing-indicator">
                        <span></span><span></span><span></span>
                      </div>
                    </div>
                  </div>
                </div>

              </div>
            </div>
            <!-- Main chat footer -->
            <div class="main-chat-footer" style="width: 48.5vw ; ">
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
        <!-- File Summary Section -->
<div class="col-md-3" style="width: 50vw; height: 80vh;">
  <div class="file-summary border p-3" style="height: 80vh;">
    <h4>File Summary</h4>
    <!-- This is where the fileSummary will be rendered -->
    <div v-if="fileSummary.length > 0">
      <div v-for="summary in fileSummary" :key="summary" v-html="summary" class="summary-item"></div>
    </div>
    <div v-else>No file summary</div>
  </div>
</div>

<!-- Chunks Section -->
<div class="col-md-3" style="width: 50vw; height: 80vh;">
  <div class="file-summary border p-3" style="height: 80vh;">
    <h4>Chunks</h4>
    <div v-if="!chunksSummary || chunksSummary.length === 0">No chunks</div>
    <div v-if="chunksSummary" v-for="chunk in chunksSummary" v-html="chunk" class="chunk-item"></div>
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
      fileSummary: [],
      chunksSummary: [],
      addQuery: "",
      lastMessage: "",
      botTyping: false,  // Track when the bot is "typing"
      isSending: false  // Flag to prevent duplicate submissions
    };
  },
  methods: {
    processChunks(result) {
  let text = ''; // Variable to store the text of the item with the max score

  result.forEach((item) => {
    // Find max of item.score
    if (!text || item.score > result.maxScore) {
      text = item.text;
      result.maxScore = item.score; // Keep track of the max score in result object
    }
  });

  

  // Replace empty lines in the text with '\n'
  text = text.replace(/^\s*$/gm, '\n');

  console.log("Summary: ", this.fileSummary);
  console.log("Text: ", text);
  // return this.fileSummary[0] + text;
  return text
},

scrollToBottom() {
        this.$nextTick(() => {
            const chatBody = this.$refs.chatBody;
            if (chatBody) {
                chatBody.scrollTo({
                    top: chatBody.scrollHeight,
                    behavior: 'smooth' // Enables smooth scrolling
                });
            }
        });
    },

    async sendMessage() {
      // Check if already sending a message
      if (this.isSending) return;


      const message = this.userMessage.trim(); // Trim whitespace
      this.lastMessage = message;
      if (message) {
        this.isSending = true; // Set the flag to true


        // Add user message to the chat
        this.messages.push({ sender: "user", text: message });
        this.userMessage = ""; // Clear input field after sending message

         // Clear previous data before new request
        this.fileSummary = [];
        this.chunksSummary = [];

        // Call scrollToBottom to scroll to the latest message
        this.scrollToBottom();

        // Show bot typing indicator
        this.botTyping = true;



       
        // upload message
        try {
            const response = await axios.post(
              "http://192.168.0.193:8000/api/query",
              {
                query: message
              }
            );
            this.fileSummary.push(response.data.context);
            await response.data.chunks.forEach((chunk) => {
              this.chunksSummary.push(chunk);
              this.fileSummary.push(
  `<span style="color: blue;">chunk id: ${chunk.chunk_id}</span>`,
  `<span style="color: blue;">doc name: ${chunk.doc_name}</span>`,
  `<span style="color: blue;">doc uuid: ${chunk.doc_uuid}</span>`,
  `<span style="color: blue;">score: ${chunk.score}</span>`,
  `<span>text: ${chunk.text}</span>`
);
            })
            
            console.log("Response received:", response.data);
            console.log("Concatenated: ", this.processChunks(this.chunksSummary));
            this.addQuery = "";
            this.addQuery = {
              "context" : this.processChunks(this.chunksSummary),
              "conversation" : [],
              "query" : this.lastMessage
            }

        

            try {
            const response = await axios.post(
              "http://192.168.0.193:8000/api/query",
              this.addQuery
            );
            console.log("response", response)
            let text = ''; // Variable to store the text of the item with the max score

            response.data.chunks.forEach((item) => {
              // Find max of item.score
              if (!text || item.score > response.data.maxScore) {
                text = item.text;
                response.data.maxScore = item.score; // Keep track of the max score in result object
              }
              console.log("The answer is:",text);
               // Bot response 
     this.messages.push({ sender: "bot", text: this.cleanText(text) });
            })

            // Call scrollToBottom to scroll to the latest message
          // this.scrollToBottom();

             // Hide typing indicator after receiving response
              // this.botTyping = false; // Bot finished typing
              
          } catch (error) {
            // Handle any errors
            console.error("Error:", error);
            alert("Failed to fetch response.");
          } finally {
      this.isSending = false; // Reset the sending flag
      this.botTyping = false; // Hide typing indicator after response
      this.scrollToBottom(); // Scroll to latest message
    }
    


          } catch (error) {
            // Handle any errors
            console.error("Error:", error);
            alert("Failed to fetch response.");
          }
      }
    },
    cleanText(text) {
  // Define the pattern for special characters to remove, keep alphanumerics and basic punctuation.
  const cleanedText = text.replace(/[^a-zA-Z0-9.,!?;:'"()\s-]/g, "");
  
  return cleanedText;
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
              "http://192.168.0.193:8000/api/import",
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
.main-chat-header {
  background-color: #0033CC;
  width:48.4vw;
}

h6 { 
  color: rgba(187,242,255,255);
}

small{
  color: rgba(187,242,255,255);
}

/* .nav-tabs .nav-link {
  cursor: pointer;
  background: rgb(237, 237, 237,1);
  backdrop-filter: blur(10px);
} */

.nav-tabs .nav-link.tab-highlight {
  background-color: #007FFF; /* Blue color for active tab */
  color: white; /* White text on active tab */
  font-weight: bold; /* Optional: Make text bold for emphasis */
}

.nav-tabs .nav-link {
  cursor: pointer;
  background: rgb(237, 237, 237, 1); /* Default background color */
  backdrop-filter: blur(10px);
  color: black;
}

.nav-tabs .nav-link.active {
  border-color: #007FFF; /* Blue border for active tab */
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
  background-color: rgba(255,255,255,255);
}

.typing-indicator {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  height: 20px;
}

.typing-indicator span {
  display: inline-block;
  width: 8px;
  height: 8px;
  margin: 0 2px;
  background-color: #ccc;
  border-radius: 50%;
  opacity: 0;
  animation: typing 1.5s infinite;
}

.typing-indicator span:nth-child(1) {
  animation-delay: 0.2s;
}
.typing-indicator span:nth-child(2) {
  animation-delay: 0.4s;
}
.typing-indicator span:nth-child(3) {
  animation-delay: 0.6s;
}

@keyframes typing {
  0% {
    opacity: 0;
  }
  50% {
    opacity: 1;
  }
  100% {
    opacity: 0;
  }
}

.main-chat-body .media.flex-row-reverse .main-msg-wrapper {
    background-color: #007FFF;
    color: #fff;
}

.main-msg-wrapper {
  background-color: rgba(239,242,247,255);
  color: rgba(39,45,59,255);
}


</style>

now make it when I select the chat section it the bar at the chat part is highlighted with a color blue when the document upload section is selected the bar part of the select section it changes to blue also

