<template>
  <div class="container">
    <div class="configuration" style="padding-right:1em">
      <label for="gpt-role">GPT's Role:</label>
      <textarea style="height: 5em" type="text" id="gpt-role" v-model="gptRole" />
      <label for="API-key">API-Key:</label>
      <textarea style="height: 5em"  type="text" id="API-key" v-model="apiKey" />
    </div>
    <div style="flex: 1;display:flex;flex-direction: column;">
      <div class="chat-window">
        <div v-for="prompt in display" class="message">
          <span class="message-role">{{ prompt.role }}</span>
          <div class="message-content-wrapper">
            <div class="message-content">{{ prompt.content }}</div>
            <div class="message-face-wrapper">
              <img :src="prompt.faceUrl" class="generated-face" />
              <div class="face-text" style="white-space: pre-line">{{ prompt.faceText }}</div>
            </div>
          </div>
        </div>
      </div>
      <div class="input-wrapper">
        <input type="text" v-model="inputText" />
        <button @click="fetchData">Call API</button>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, watch } from "vue";
import axios from "axios";

export default {
  setup() {
    const inputText = ref("");
    const display = ref([]);
    const gptRole = ref("You are a grumpy old man");
    const facePrompt = ref("photo. the head of pierce brosnan in james bond")
    const showFace = ref(true);
    const prompts = ref([{role: "system", content: gptRole.value}]);
    const gpt3Endpoint = "https://api.openai.com/v1/chat/completions";
    const apiKey = ref('');

    watch(gptRole, () => {
      prompts.value[0].content = gptRole.value;
    });

    async function fetchData() {
      try {
        prompts.value.push(
          {
            role: "user",
            content: inputText.value,
          })
        display.value.push(
          {
            role: "User",
            content: inputText.value,
          })
        const call = await axios.post(
          gpt3Endpoint,
          {
            model: "gpt-3.5-turbo",
            messages: prompts.value,
            max_tokens: 500, // Adjust the number of tokens to return as per your requirement
          },
          {
            headers: {
              'Content-Type': 'application/json',
              'Authorization': `Bearer ${apiKey.value}`,
            },
          }
        );

        console.log(call.data.choices[0].message.content);
        prompts.value.push(
        {
          role: "assistant",
          content: call.data.choices[0].message.content,
        });
        display.value.push(
          {
            role: "Assistant",
            content: call.data.choices[0].message.content
          })
      } catch (error) {
        console.log(error);
      }
    }

    return {
      inputText,
      display,
      gptRole,
      facePrompt,
      showFace,
      apiKey,
      fetchData,
    };
  },
};
</script>
<style>
  html, body {
    height: 100%;
    margin: 0;
    
  }
  #app {
    display: flex;
    height: 100%;
  }
  .configuration {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
    gap: 0.5rem;
    margin-bottom: 1rem;
  }
  .container {
    max-width: 800px;
    margin: 0 auto;
    display: flex;
    flex-direction: row;
    height: calc(100% - 2em);;
    padding: 1em;
  }
  .chat-window {
    flex: 1;
    border: 1px solid #ccc;
    border-radius: 4px;
    padding: 1rem;
    overflow-y: auto; 
    max-height: 100%; 
    background-color: #f8f8f8;
    margin-bottom: 1rem;

  }


  .input-wrapper {
    display: flex;
  }
  input[type="text"] {
    flex: 1;
    padding: 0.5rem;
    border-radius: 4px;
    border: 1px solid #ccc;
    margin-right: 0.5rem;
  }
  button {
    padding: 0.5rem 1rem;
    border: none;
    border-radius: 4px;
    background-color: #007bff;
    color: #fff;
    cursor: pointer;
  }
  .message {
  display: flex;
  flex-direction: column;
  align-items: stretch;
  margin-bottom: 1rem;
}

.message-content-wrapper {
  display: flex;
  flex-direction: row;
}

.message-face-wrapper {
  display: flex;
  flex-direction: column;
  align-items: center;
  max-width: 8em;
}

.generated-face {
  width: 50%;
  height: auto;
  margin-left: 1rem;
}

.face-text {
  font-size: 0.8rem;
  color: #777;
  margin-top: 0.25rem;
  text-align: center;
}
  .message-role {
    font-weight: bold;
    margin-bottom: 0.25rem;
  }
  .message-content {
    padding: 0.5rem 1rem;
    border-radius: 1rem;
    background-color: #e0e0e0;
    display: inline-block;
  }
</style>