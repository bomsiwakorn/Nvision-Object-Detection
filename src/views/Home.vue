<template>
  <div id="home">
    <v-container grid-list-xs>
      <v-card elevation="0" class="mt-3">
        <v-container fluid>
          <v-card-title primary-title>
            Nipa.Cloud Nvision's Object Detection
          </v-card-title>
          <v-divider class="mx-4 mb-8"></v-divider>
          <v-col cols="12" class="px-4">
            <v-row>
              <v-col cols="12" md="8">
                <div class="wrapper-upload-image" v-if="mode == 1">
                  <input
                    type="file"
                    accept="image/jpeg/png"
                    @change="handleImageFile"
                  />
                  <div v-if="!imageInputConvertBase64">
                    <img src="@/assets/upload.svg" alt="Upload" />
                    <span>Upload Image</span>
                  </div>
                  <img class="imagePreview" :ref="'image'" />
                </div>
                <div class="wrapper-upload-image" v-if="mode == 2">
                  <div class="wrapper-web-cam">
                    <img class="imageCapture" :ref="'imageCapturePreview'" />
                    <vue-web-cam
                      ref="webcam"
                      :device-id="deviceId"
                      width="100%"
                      @cameras="onCameras"
                      @camera-change="onCameraChange"
                      v-if="!imageCapture"
                    />

                      <div class="text-center">
                        <v-btn
                        class="mt-4"
                        @click.prevent="onCapture()"
                        color="#e01a70"
                        dark
                        outlined
                        rounded
                        v-if="!imageCapture"
                      >
                        Capture
                      </v-btn>
                        <v-btn
                        class="mt-4"
                        @click.prevent="onCaptureAgain"
                        color="#e01a70"
                        dark
                        outlined
                        rounded
                        v-if="imageCapture"
                      >
                        Capture Again
                      </v-btn>
                      </div>
   
                  </div>
                </div>
              </v-col>
              <v-col cols="12" md="4">
                <v-card-text class="font-weight-medium np_fs18 px-0 px-md-4">
                  Input Mode
                </v-card-text>
                <v-divider class="ml-0 ml-md-4 mb-6"></v-divider>
                <div class="wrapper-select-input-btn ml-0 ml-md-4">
                  <v-btn
                    @click.prevent="changeModeUpload"
                    large
                    elevation="0"
                    class="my-4"
                    dark
                    block
                    :outlined="mode === 1 ? false : true"
                    rounded
                    :color="mode === 1 ? '#e01a70' : '#777'"
                  >
                    <v-icon left>mdi-image</v-icon>
                    Upload Mode
                  </v-btn>
                  <v-btn
                    @click.prevent="changeModeWebCam"
                    large
                    elevation="0"
                    class="my-4"
                    dark
                    block
                    :outlined="mode === 2 ? false : true"
                    rounded
                    :color="mode === 2 ? '#e01a70' : '#777'"
                  >
                    <v-icon left>mdi-camera</v-icon>
                    Camera Mode
                  </v-btn>
                </div>
                <div class="wrapper-detect-btn ml-0 ml-md-4 mt-9">
                  <v-btn
                    @click.prevent="detectSubmit"
                    :loading="loadingSubmit"
                    x-large
                    block
                    rounded
                    dark
                    class="submit-detect-btn"
                    >Detect</v-btn
                  >
                </div>
              </v-col>
            </v-row>
          </v-col>
        </v-container>
      </v-card>

      <v-card class="mt-4 mb-10" elevation="0">
        <v-container fluid>
          <v-card-title primary-title> Result </v-card-title>
          <v-divider class="mx-4 mb-8"></v-divider>
          <div class="px-4">
            <div class="container-image-result" id="container-image-result">
              <img class="imageResult" :ref="'imageResult'" />
            </div>
          </div>
          <div class="text-center" v-if="checkResult">
            <img width="300px" src="@/assets/image.svg" alt="">
          </div>
        </v-container>
      </v-card>
    </v-container>
    <Footer />
  </div>
</template>

<script>
import axios from "axios";
import { WebCam } from "vue-web-cam";
import Footer from "@/components/Footer"
import Swal from "sweetalert2"
export default {
  name: "Home",
  components: {
    "vue-web-cam": WebCam,
    Footer
  },
  data() {
    return {
      deviceId: null,
      devices: [],
      mode: 1,
      openCamera: false,
      imageInputBase64: null,
      imageCapture: null,
      imageInputConvertBase64: null,
      loadingSubmit: false,
      checkResult: true
    };
  },
  watch: {
    devices() {
      if (this.devices[0]) {
        this.deviceId = this.devices[0].deviceId;
      }
    },
  },
  methods: {
    changeModeUpload() {
      if (this.mode === 2) {
        this.mode = 1;
        this.imageCapture = null
        this.imageInputConvertBase64 = null
      }
    },
    changeModeWebCam() {
      if (this.mode === 1) {
        this.onCameraChange();
        this.mode = 2;
        this.imageInputConvertBase64 = null
      }
    },
    onCapture() {
      this.imageCapture = this.$refs.webcam.capture();
      this.$refs.imageCapturePreview.src = this.imageCapture
      const imageCaptureSplit = this.imageCapture.split(",");
      this.imageInputConvertBase64 = imageCaptureSplit[1];
    },
    onCaptureAgain() {
      this.imageCapture = null
      this.imageInputConvertBase64 = null
      this.$refs.imageCapturePreview.src = ''
      this.onCameraChange()
    },
    onCameras(cameras) {
      this.devices = cameras;
    },
    onCameraChange(deviceId) {
      this.deviceId = deviceId;
    },
    handleImageFile(event) {
      return new Promise((resolve, reject) => {
        const imageFile = event.target.files;
        const reader = new FileReader();
        reader.readAsDataURL(imageFile[0]);
        reader.onload = (e) => {
          this.$refs.image.src = reader.result;
          this.imageInputBase64 = reader.result;
          let base64Url = reader.result.split(",");
          this.imageInputConvertBase64 = base64Url[1];
        };
        reader.onerror = (error) => reject(error);
      });
    },
    serviceNvisionDetectObject(API_KEY, imageBase64) {
      return new Promise(async (resolve, reject) => {
        try {
          const response = await axios({
            method: "post",
            url: "https://nvision.nipa.cloud/api/v1/object-detection",
            data: { raw_data: imageBase64 },
            headers: {
              "Content-Type": "application/json",
              Authorization: `ApiKey ${API_KEY}`,
            },
          });
          return resolve(response.data);
        } catch (error) {
          return reject(error);
        }
      });
    },
    async detectSubmit() {
      if (this.imageInputConvertBase64) {
        this.loadingSubmit = true;
        this.clearResultObject()
        try {
          const API_KEY = "cdb29f355cb4059995e05420dc8d963f657898bf3a5f2f5e7a88c58279f5e4a0a1c4c4cf874594b42e413fc45c425425ac";
          const result = await this.serviceNvisionDetectObject(
            API_KEY,
            this.imageInputConvertBase64
          );
          if (!result.detected_objects) {
            this.loadingSubmit = false;
            return Swal.fire({
              icon: 'warning',
              text: 'There are no objects to display.',
              toast: true,
              position: 'bottom-end',
              showConfirmButton: false,
              timer: 5000,
              width: '390px'
            })
          }
          const detectedObjects = result.detected_objects;
          const parents = [...new Set(detectedObjects.map(x => x.parent))]
          const newDetectedObjects = []
          parents.forEach(parent => {
            const randomColor = Math.floor(Math.random()*16777215).toString(16);
            detectedObjects.map(data => {
              if (parent === data.parent) {
                data.color = `#${randomColor}`
                newDetectedObjects.push(data)
              }
            })
          })
          const containerBox = document.getElementById("container-image-result");
          for (const index in newDetectedObjects) {
            let div = document.createElement("div");
            let child1 = document.createElement("div");
            let child2 = document.createElement("div");
            div.className = "box"
            child1.className = "wrapper-text"
            child2.className = "text"
            div.style.position = "absolute";
            div.style.border = `2px solid ${detectedObjects[index].color}`;
            div.style.left = detectedObjects[index].bounding_box.left + "px";
            div.style.right = detectedObjects[index].bounding_box.right + "px";
            div.style.bottom = detectedObjects[index].bounding_box.bottom + "px";
            div.style.top = detectedObjects[index].bounding_box.top + "px";
            div.style.width =
              detectedObjects[index].bounding_box.right -
              detectedObjects[index].bounding_box.left +
              "px";
            div.style.height =
              detectedObjects[index].bounding_box.bottom -
              detectedObjects[index].bounding_box.top +
              "px";
            child1.style.position = "relative"
            child2.style.position = "absolute"
            child2.style.left = "0"
            child2.style.top = "0"
            child2.style.background = `${detectedObjects[index].color}e0`
            child2.style.whiteSpace = "nowrap"
            child2.style.color = "#fff"
            child2.innerHTML = `name: ${detectedObjects[index].name} | parent: ${detectedObjects[index].parent} | confidence: ${Math.round(detectedObjects[index].confidence * 100)}%`
            containerBox.appendChild(div);
            div.appendChild(child1);
            child1.appendChild(child2);
          }
          this.loadingSubmit = false;
          this.$refs.imageResult.src = this.mode === 1 ? this.imageInputBase64 : this.imageCapture;
          this.checkResult = false
        } catch (error) {
          this.loadingSubmit = false;
          console.log(error);
        }
      }
    },
    clearResultObject() {
      let allBox = document.querySelectorAll(".box"); 
      this.$refs.imageResult.src = ''
      if (allBox.length > 0) {
        allBox.forEach((elem) => elem.remove());
      }
    }
  },
};
</script>
<style scoped>
.submit-detect-btn {
  background: linear-gradient(
    150deg,
    rgba(224, 26, 112, 1) 0%,
    rgba(85, 43, 81, 1) 100%
  );
}
.wrapper-upload-image {
  width: 100%;
  background: #f4f4f4;
  padding: 1rem;
  position: relative;
  display: grid;
  grid-template-columns: 100%;
  justify-content: center;
  align-items: center;
  border-radius: 5px;
}
.wrapper-upload-image div {
  display: grid;
  justify-content: center;
  align-items: center;
}
.wrapper-upload-image input {
  width: 100%;
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  opacity: 0;
  cursor: pointer;
}
.wrapper-upload-image span {
  padding: 10px;
  text-align: center;
  font-weight: 500;
  color: #e01a70;
}
.imagePreview, .imageCapture {
  width: 100%;
}
.container-image-result {
  width: 100%;
  position: relative;
  overflow-x: scroll;
}

</style>
