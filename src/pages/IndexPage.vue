<template>
  <div>
    <div id="my-form">
      <div class="loader-wrapper">
        <div class="loader"></div>
      </div>
      <span class="text first-text"></span>
      <span class="text sec-text"></span>
    </div>
    <div class="container">
      <span class="mynew"></span>
      <span class="mynewtwo"></span>
    </div>
    <div class="full"></div>
    <q-page padding>
      <div class="main">
        <q-form class="form" @submit.prevent="generateResponse">
          <q-input
            rounded
            outlined
            v-model="input"
            class="custom-input-style"
            color="black"
            style="width: 40%"
            label="Insert More Information..."
          >
            <template v-slot:append>
              <q-avatar>
                <img
                  src="https://cdn.discordapp.com/attachments/997271660900126801/1098598628915957810/coullax_ai_bot_logo_smaller.png"
                />
              </q-avatar>
            </template>
          </q-input>
          <q-btn class="submit" type="submit" label="" />
        </q-form>
        <div class="chatLog">
          <q-card v-for="(item, index) in log" :key="index" class="my-card">
            <q-card-section>
              <div id="new" class="text-h6">User: {{ item.input }}</div>
              <transition name="type">
                <div
                  id="text"
                  class="text-h6 bot-message"
                  :class="{ typing: item.typing }"
                >
                  <template v-if="!item.typing">
                    {{ item.response }}
                  </template>
                </div>
              </transition>
            </q-card-section>
          </q-card>
        </div>
      </div>
    </q-page>
  </div>
</template>

<script>
import { defineComponent } from "vue";
import { Configuration, OpenAIApi } from "openai";
import VueTypedJs from "vue-typed-js";

export default defineComponent({
  name: "IndexPage",
  data() {
    return {
      OPENAI_API_KEY: "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",
      input: "",
      response: "",
      log: [],
      messages: [],
      loading: false,
    };
  },
  methods: {
    async completionCall(input) {
      const prefixText = "information";
      const combinedInput = prefixText + input;
      this.messages.push({ role: "user", content: combinedInput });

      const configuration = new Configuration({
        apiKey: process.env.OPENAI_API_KEY || this.OPENAI_API_KEY,
      });
      const openai = new OpenAIApi(configuration);

      const completion = await openai.createChatCompletion({
        model: "gpt-3.5-turbo",
        messages: this.messages,
      });
      this.messages.push({
        role: completion.data.choices[0].message.role,
        content: completion.data.choices[0].message.content,
      });
      this.response = completion.data.choices[0].message.content;
      console.log(completion.data.choices[0].message.content);
    },
    async generateResponse() {
      this.submitted = true;
      this.loading = true;

      // Show user input immediately
      this.log.unshift({
        input: this.input + "",
        response: "",
      });
      this.input = "";

      //loading animation
      const form = document.getElementById("my-form");
      const loader = document.querySelector(".loader");

      form.addEventListener("submit", function (event) {
        event.preventDefault(); // prevent form from submitting normally
        loader.style.display = "block"; // show the loader
        // submit the form data
      });

      function toggleLoader(event) {
        if (event.key === "Enter") {
          loader.style.display = "block";
        } else {
          loader.style.display = "none";
        }
      }

      loader.style.display = "none"; // hide the loader initially

      document.addEventListener("keydown", toggleLoader);

      // Typing animation
      const mynew = document.querySelector(".mynew");
      const mynewtwo = document.querySelector(".mynewtwo");
      const responseContainer = document.querySelector(".response-container");
      const responseText = document.querySelector(".response-text");
      const texts = ["Thinking...", "I'm processing...", "Let me see..."];
      let currentTextIndex = 0;
      let currentCharacterIndex = 0;
      let currentText = texts[currentTextIndex];
      const typingSpeed = 100;
      const pauseDuration = 2000;
      let timeout;

      function typeText() {
        if (currentTextIndex === 2) {
          // if we are on the last text ("Let me see...")
          // stop the typing animation and show the secret number
          clearTimeout(timeout);
          displayBotResponse(
            "Your secret number is: " + generateSecretNumber()
          );
        } else if (currentCharacterIndex < currentText.length) {
          if (currentTextIndex === 0) {
            mynew.textContent += currentText[currentCharacterIndex];
          } else if (currentTextIndex === 1) {
            mynewtwo.textContent += currentText[currentCharacterIndex];
          }
          currentCharacterIndex++;
          timeout = setTimeout(typeText, typingSpeed);
        } else {
          setTimeout(eraseText, pauseDuration);
        }
      }

      function eraseText() {
        if (currentCharacterIndex > 0) {
          if (currentTextIndex === 0) {
            mynew.textContent = currentText.substring(
              0,
              currentCharacterIndex - 1
            );
          } else if (currentTextIndex === 1) {
            mynewtwo.textContent = currentText.substring(
              0,
              currentCharacterIndex - 1
            );
          }
          currentCharacterIndex--;
          timeout = setTimeout(eraseText, typingSpeed);
        } else {
          currentTextIndex = (currentTextIndex + 1) % texts.length;
          currentText = texts[currentTextIndex];
          timeout = setTimeout(typeText, typingSpeed);
        }
      }

      function displayBotResponse(response) {
        if (response.includes("567")) {
          // Display the secret key without the typing animation
          responseText.textContent = "";
          responseContainer.style.display = "flex";
          mynew.style.display = "none";
          mynewtwo.style.display = "none";
          clearTimeout(timeout);
          setTimeout(() => {
            responseContainer.style.display = "none";
            mynew.style.display = "block";
            mynewtwo.style.display = "block";
            location.reload(); // Reload the page after the response is displayed
          }, 50000);
        } else {
          // Display the response with the typing animation
          responseText.textContent = response;
          responseContainer.style.display = "flex";
          mynew.style.display = "none";
          mynewtwo.style.display = "none";
          clearTimeout(timeout);
          setTimeout(() => {
            responseContainer.style.display = "none";
            mynew.style.display = "block";
            mynewtwo.style.display = "block";
            location.reload(); // Reload the page after the response is displayed
          }, 50000);
        }
      }

      function handleUserResponse() {
        displayBotResponse("This is a bot response.");
        clearTimeout(timeout);
      }
      function showLoading() {
        document.getElementById("loading").style.display = "block";
      }

      typeText();

      // Bot response
      const secretKey = 567; // Generate a secret key
      console.log(`Secret key: ${secretKey}`); // Display the secret key in the console

      await this.completionCall(this.log[0].input).then(() => {
        this.log[0].response = this.response;
        this.loading = false;
        var responseText = this.response;
        var words = responseText.split(" ");
        var i = 0;
        document.getElementById("text").innerHTML = ""; // clear previous response
        function typing() {
          if (i < words.length) {
            let word = words[i];
            let j = 0;
            let intervalId = setInterval(() => {
              if (j < word.length) {
                document.getElementById("text").innerHTML += word[j];
                j++;
              } else {
                document.getElementById("text").innerHTML += " ";
                clearInterval(intervalId);
                i++;
                setTimeout(typing, 150); // delay between words
              }
            }, 50); // delay between letters
          } else {
            // Display secret key and hide it after 5 seconds
            document.getElementById("text").innerHTML += secretKey;
            setTimeout(() => {
              const responseEl = document.getElementById("text");
              responseEl.innerHTML = responseEl.innerHTML.replace(
                secretKey,
                ":)"
              );
            }, 100);
          }
        }
        typing();
      });
    },
  },
});
</script>

<style>
.my-card {
  margin-top: 6%;
}

@media only screen and (max-width: 768px) {
  .my-card {
    margin-top: 15%;
  }
}

@media only screen and (max-width: 996px) {
  .my-card {
    margin-top: 15%;
  }
}

.full {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 110px;
  z-index: 9;
  background-color: #ffffff;
  overflow: hidden;
}

.submit {
  position: fixed;
  left: 70.5%;
  background-image: url("submit.png");
  background-size: 70px 70px;
  transform: translateX(-50%);
  font-size: 1.6rem;
  background-color: #ffffff;
  padding: 23px 20px 48px 50px;
  align-items: center;
  justify-content: space-between;
  border: none;
  border-radius: 100%;
  overflow: hidden;
  transition: all 0.3s;
  z-index: 999;
  margin-top: 0 auto;
}

.submit:hover {
  background-color: #f5eb7e;
}
.main {
  max-width: 1200px;
  margin: 0 auto;
}

.image-container {
  position: fixed;
  bottom: -300px;
  left: 0px;
  overflow: visible;
  width: 100px;
  height: 100px;
}

@media only screen and (max-width: 768px) {
  .submit {
    left: 80%;
    transform: translateX(-50%);
  }
}

.custom-input-style {
  position: fixed;
  left: 47%;
  z-index: 999;
  transform: translateX(-50%);
  margin-top: 7px;
}

.form {
  display: flex;
  width: 100%;
}

.container {
  position: fixed;
  top: 5%;
  left: 50%;
  width: 100%;
  height: auto;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  z-index: 999;
  transform: translate(-50%, 50%);
  font-family: "Montserrat", sans-serif;
  font-size: 15px;
  color: #000000;
  border-radius: 20px;
  padding: 30px;
}

#my-form {
  position: absolute;
  top: 40%;
  left: 50%;
  transform: translate(-50%, -50%);
}

.loader-wrapper {
  display: none;
}

#text {
  background-color: #f8f8f8;
  border-radius: 10px;
  padding: 40px;
  color: #333;
  font-size: 18px;
  text-align: left;
  margin-top: 50px;
  margin-bottom: 60px;
  margin-right: 270px;
  font-family: "Poppins", sans-serif;
  font-weight: 600;
  box-shadow: 0px 2px 5px rgba(0, 0, 0, 0.1);
  max-width: 70%;
  bottom: 0;
  right: 30px;
}

@media screen and (max-width: 768px) {
  #text {
    max-width: 90%;
    margin-right: 20px;
    margin-left: 20px;
  }
}

@media screen and (max-width: 480px) {
  #text {
    font-size: 16px;
    padding: 20px;
  }
}

#text:before {
  content: "";
  position: absolute;
  top: 80%;
  left: -20px;
  margin-top: -10px;
  border-style: solid;
  border-width: 10px 20px 10px 0;
  border-color: transparent #f8f8f8 transparent transparent;
}

#text:after {
  content: "";
  position: absolute;
  top: 50%;
  right: -20px;
  margin-top: -10px;
  border-style: solid;
  border-width: 10px 0 10px 20px;
  border-color: transparent transparent transparent #f8f8f8;
}

#new {
  padding: 10px 15px;
  border-radius: 20px;
  background-color: #bebebe;
  box-shadow: 0px 2px 10px rgba(0, 0, 0, 0.1);
  font-family: "Segoe UI", sans-serif;
  font-size: 16px;
  font-weight: 600;
  line-height: 1.5;
  color: #000000;
  margin-left: 80%;
}

@keyframes type {
  0% {
    width: 0%;
  }
  100% {
    width: 100%;
  }
}

@keyframes type {
  0% {
    opacity: 0;
    transform: translateX(-10px);
  }
  100% {
    opacity: 1;
    transform: translateX(0px);
  }
}

.loader {
  animation: rotate 1s infinite;
  height: 50px;
  width: 50px;
}

.loader:before,
.loader:after {
  border-radius: 50%;
  content: "";
  display: block;
  height: 20px;
  width: 20px;
}
.loader:before {
  animation: ball1 1s infinite;
  background-color: #000000;
  box-shadow: 30px 0 0 #f8b334;
  margin-bottom: 10px;
}
.loader:after {
  animation: ball2 1s infinite;
  background-color: #f8b334;
  box-shadow: 30px 0 0 #000000;
}

.loader-wrapper {
  display: flex;
  align-items: center;
  justify-content: center;
}

@keyframes rotate {
  0% {
    -webkit-transform: rotate(0deg) scale(0.8);
    -moz-transform: rotate(0deg) scale(0.8);
  }
  50% {
    -webkit-transform: rotate(360deg) scale(1.2);
    -moz-transform: rotate(360deg) scale(1.2);
  }
  100% {
    -webkit-transform: rotate(720deg) scale(0.8);
    -moz-transform: rotate(720deg) scale(0.8);
  }
}

@keyframes ball1 {
  0% {
    box-shadow: 30px 0 0 #f8b334;
  }
  50% {
    box-shadow: 0 0 0 #f8b334;
    margin-bottom: 0;
    -webkit-transform: translate(15px, 15px);
    -moz-transform: translate(15px, 15px);
  }
  100% {
    box-shadow: 30px 0 0 #f8b334;
    margin-bottom: 10px;
  }
}

@keyframes ball2 {
  0% {
    box-shadow: 30px 0 0 #000000;
  }
  50% {
    box-shadow: 0 0 0 #000000;
    margin-top: -20px;
    -webkit-transform: translate(15px, 15px);
    -moz-transform: translate(15px, 15px);
  }
  100% {
    box-shadow: 30px 0 0 #000000;
    margin-top: 0;
  }
}
</style>
