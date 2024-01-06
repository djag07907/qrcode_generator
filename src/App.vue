<template>
  <div>
    <header>
      <span>QR Code Generator</span>
    </header>
    <div class="container">
      <div class="left">
        <textarea
          class="user-input-field"
          placeholder="Enter your text here"
          v-model="text"
        ></textarea>
      </div>
      <div class="right">
        <template v-if="text && text.length > 0">
          <div class="qrcode-box">
            <div
              v-if="text && text.length > 0"
              class="qrcode"
              ref="templateReference"
            >
              <vue-qrcode
                :value="text"
                :options="{
                  width: 200,
                }"
              ></vue-qrcode>
              <img
                v-if="useLogo && uploadedImgUrl && uploadedImgUrl.length !== 0"
                class="qrcode__image"
                :src="uploadedImgUrl"
                alt="business logo"
              />
            </div>
          </div>
          <div>
            <input type="checkbox" id="toggle-logo" v-model="useLogo" />
            <label for="toggle-logo"> Add logo to QR code </label>
          </div>
          <input
            v-if="useLogo"
            type="file"
            @change="handleLogoUpload"
            accept="image/*"
          />
          <button @click="downloadQRCode" class="download-button">
            <span v-if="!downloading">download code as png</span>
            <span v-else class="spinner"></span>
          </button>
          <div v-if="error" class="error-message">{{ error }}</div>
        </template>
        <div v-else class="placeholder">
          <p>Enter some text on the left to generate a code</p>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import VueQrcode from "@chenfengyuan/vue-qrcode";
import { toPng } from "html-to-image";
import { ref } from "vue";

const text = ref("");
const downloading = ref(false);
const templateReference = ref(null);
const useLogo = ref(false);
const uploadedImgs = ref([]);
const uploadedImgUrl = ref("");
const error = ref("");

const downloadQRCode = () => {
  if (templateReference.value === null || !text.value) {
    error.value = "Please enter some text before downloading the QR code.";
    return;
  }
  downloading.value = true;
  toPng(templateReference.value, { cacheBust: true })
    .then((dataUrl) => {
      const link = document.createElement("a");
      link.download = "my-qrcode.png";
      link.href = dataUrl;
      link.click();
      downloading.value = false;
      error.value = "";
    })
    .catch((err) => {
      console.log(err, templateReference.value);
      downloading.value = false;
      error.value = "An error occurred while generating the QR code.";
    });
};

const handleLogoUpload = ($event) => {
  const input = $event.target;
  uploadedImgs.value = input.files;
  if (uploadedImgs.value.length === 0) {
    return;
  }
  const reader = new FileReader();
  reader.addEventListener(
    "load",
    () => {
      uploadedImgUrl.value = reader.result;
    },
    false
  );
  reader.readAsDataURL(uploadedImgs.value[0]);
};
</script>
