<template>
  <f7-page>
    <f7-navbar title="Lapor" back-link="Back">
      <f7-nav-right>
        <f7-link icon-ios="f7:save" icon-md="material:check" @click="submitFile"></f7-link>
      </f7-nav-right>
    </f7-navbar>
    <f7-block-title>Unggah Foto</f7-block-title>
      <f7-block>
        <center>
          <div class="image-upload" raised @click="$refs.actionsOneGroup.open()">
              <f7-icon material="add_a_photo" size="100px" :class="{first: showPreview}"></f7-icon>
              <img id="myimg" ref="myimg" style="max-width: 200px; max-height: 200px;"
              :class="{preview: showPreview}"
              :src="imagePreview"
              v-show="showPreview"/>
              <span>{{filename}}</span>
            <f7-actions ref="actionsOneGroup">
              <f7-actions-group>
                <f7-actions-label bold>Unggah Foto</f7-actions-label>
                <f7-actions-button v-if="camera" @click="openGallery"><f7-icon material="collections"></f7-icon>Galeri</f7-actions-button>
                <f7-actions-button v-else @click="openGalleryWeb"><f7-icon material="collections"></f7-icon>Galeri</f7-actions-button>
                <f7-actions-button @click="takePicture"><f7-icon material="camera_alt"></f7-icon>Kamera</f7-actions-button>
                <f7-actions-button color="red"><f7-icon material="cancel"></f7-icon>Cancel</f7-actions-button>
              </f7-actions-group>
            </f7-actions>
            <input id="file-input" type="file" ref="file" accept="image/*" v-on:change="handleFileUpload()" required validate />
          </div>
          <!-- <div>
                <vue-json-pretty
                  :path="'res'"
                  :data="{ metadata }"
                  :options="{maxDepth: 3}">
                </vue-json-pretty>
              </div> -->
        </center>
      </f7-block>
      <f7-block-title>Kategori</f7-block-title>
      <f7-block style="font-size:12px;">
        <center>
          <f7-row>
            <f7-col>
              <div>
                <label>
                  <input type="radio" name="kat" v-model="kat" value="PJU">
                  <f7-icon material="lightbulb" f7="lightbulb" size="35px"></f7-icon>
                </label>
              </div>
              <span>Penerangan Jalan Umum (PJU)</span>
            </f7-col>
            <f7-col>
              <div>
                <label>
                  <input type="radio" name="kat" v-model="kat" value="Traffic Light">
                  <f7-icon material="traffic" size="35px"></f7-icon>
                </label>
              </div>
              <span>Traffic Light</span>
            </f7-col>
            <f7-col>
              <div>
                <label>
                  <input type="radio" name="kat" v-model="kat" value="Rambu Lalu Lintas">
                  <f7-icon material="local_parking" size="35px"></f7-icon>
                </label>
              </div>
              <span>Rambu Lalu Lintas</span>
            </f7-col>
          </f7-row>
          <f7-row style="margin-top:25px;">
            <f7-col>
              <div>
                <label>
                  <input type="radio" name="kat" v-model="kat" value="Marka Jalan">
                  <f7-icon material="texture" size="35px"></f7-icon>
                </label>
              </div>
              <span>Marka Jalan</span>
            </f7-col>
            <f7-col>
              <div>
                <label>
                  <input type="radio" name="kat" v-model="kat" value="Cermin Tikungan">
                  <f7-icon material="directions" size="35px"></f7-icon>
                </label>
              </div>
              <span>Cermin Tikungan</span>
            </f7-col>
            <f7-col>
              <div>
                <label>
                  <input type="radio" name="kat" v-model="kat" value="Perlengkapan Jalan Lainnya">
                  <f7-icon material="more" size="35px"></f7-icon>
                </label>
              </div>
              <span>Perlengkapan Jalan Lainnya</span>
            </f7-col>
          </f7-row>
        </center>
      </f7-block>
      <f7-block-title>Lokasi</f7-block-title>
      <f7-block>
        <l-map style="height: 200px;" id="map" ref="myMap" :zoom="zoom" :center="center" @click="addMarker">
          <l-tile-layer :url="url" :attribution="attribution"></l-tile-layer>
          <l-marker id="marker"
          :class="{preview: showPreview}"
          v-show="address"
          :lat-lng="marker"
          @click="removeMarker()">
            <l-popup>{{address}}</l-popup>
          </l-marker>
        </l-map>
        <span style="color: #8e8e93"><f7-icon v-show="address != null" md="material:place"></f7-icon><span style="font-size:12px;">{{address}}</span></span>
      </f7-block>
      <f7-block-title>Deskripsi</f7-block-title>
      <f7-block>
        <div class="list inline-labels no-hairlines-md">
          <ul>
            <li class="item-content item-input">
              <div class="item-inner">
                <div class="item-input-wrap">
                  <textarea v-model="desk" placeholder="Deskripsi Laporan"></textarea>
                </div>
              </div>
            </li>
          </ul>
        </div>
      </f7-block>
  </f7-page>
</template>

<script>

import {LMap, LTileLayer, LMarker, LPopup } from 'vue2-leaflet'
import * as geocoding from 'esri-leaflet-geocoder'
import * as EXIF from 'exif-js'
import VueJsonPretty from 'vue-json-pretty'
import { log } from 'util'
import axios from '../config/axiosConfig'

var geocodeService = geocoding.geocodeService()

export default {
  components: {
    LMap,
    LTileLayer,
    LMarker,
    LPopup,
    VueJsonPretty
  },
  /*
      Defines the data used by the component
    */
  data () {
    return {
      // leaflet
      zoom: 13,
      center: L.latLng(-7.484621, 112.434517),
      url: 'http://{s}.tile.osm.org/{z}/{x}/{y}.png',
      attribution: '&copy; <a href="http://osm.org/copyright">OpenStreetMap</a> contributors',
      marker: L.latLng(0, 0),
      address: null,
      district: null,

      // user
      idUser: '',

      // upload
      filename: '',
      file: '',
      showPreview: false,
      imagePreview: '',

      // axios
      kat: '',
      desk: '',
      lat: null,
      lng: null,

      // exif
      metadata: null,

      // error
      err: false,

      // camera
      camera: navigator.camera
    }
  },
  created () {
    let decode = this.$jwt.decode()
    this.idUser = decode.userId
    // Listen to Cordova's backbutton event
    document.addEventListener('backbutton', this.navigateBack, false)
  },
  methods: {
    navigateBack () {
      let app = this.$f7
      let $$ = this.$$
      // Use Framework7's router to navigate back
      let mainView = app.views.main
      if (app.views.main.router.url == '/home/tab1') {
        navigator.app.exitApp()
      } else {
        mainView.router.back('', {
          force: true
        })
      }
    },
    default () {
      console.log('defaultCallback')
    },
    openGalleryWeb () {
      document.getElementById('file-input').click()
      this.$refs.actionsOneGroup.close()
    },
    // get photo from gallery
    openGallery () {
      if (navigator.camera) {
        // Retrieve image file location from specified source
        navigator.camera.getPicture(this.setPicture, this.error, {
          quality: 70,
          destinationType: Camera.DestinationType.FILE_URI,
          sourceType: navigator.camera.PictureSourceType.SAVEDPHOTOALBUM
        })
      } else {
        this.error
      }
    },
    // Use the camera plugin to capture image
    takePicture () {
      if (navigator.camera) {
        navigator.camera.getPicture(this.setPicture, this.error, {
          quality: 70,
          destinationType: Camera.DestinationType.FILE_URI,
          sourceType: Camera.PictureSourceType.CAMERA,
          mediaType: Camera.MediaType.PICTURE,
          encodingType: Camera.EncodingType.JPEG,
          cameraDirection: Camera.Direction.BACK
        })
      } else {
        // If the navigator.camera is not available display generic error to the user.
        this.error()
      }
    },
    // Set the picture path in the data of the vue
    // this action will automatically update the view.
    setPicture (imagePath) {
      let self = this
      this.$f7.preloader.show()
      this.imagePreview = imagePath
      this.showPreview = false

      console.log(imagePath)
      // var options = new FileUploadOptions();
      //     options.fileKey = "fotoLapor";
      //     options.fileName = imagePath.substr(imagePath.lastIndexOf('/') + 1);
      //     options.mimeType = "image/jpeg";
      //     var ft = new FileTransfer();
      //     ft.upload(imagePath, "http://192.168.1.12:3000/lapor",
      //     function (result) {
      //         console.log(JSON.stringify(result));
      //     },
      //     self.error,
      //     options
      //     )

      window.resolveLocalFileSystemURL(imagePath,
        function (fileEntry) {
          // alert("got image file entry: " + fileEntry.fullPath);
          // self.filename = fileEntry.fullPath.replace("/", "");
          fileEntry.file(function (file) { // should return a raw HTML File Object
            let reader = new FileReader()
            reader.onloadend = function (e) {
              let imgBlob = new Blob([ this.result ], { type: 'image/jpeg' })
              self.file = imgBlob
            }
            reader.readAsArrayBuffer(file) // or the way you want to read it
            console.log('dari kamera: ', file)
            self.getLocation(file)
          },
          self.error)
        },
        this.error
      )
    },
    error (err) {
      self.$f7.dialog.alert(error.message, 'Terjadi Kesalahan')
      this.$f7.dialog.close()
    },
    /* Submits the file to the server */
    async submitFile () {
      try {
        let self = this

        if (this.kat == '' || this.file == '' || this.desk == '' || this.address == null) {
          self.$f7.dialog.alert('Pengisian laporan keluhan tidak lengkap', 'Terjadi Kesalahan')
        } else {
          console.log(this.file)

          /* Initialize the form data */
          let formData = new FormData()

          /* Add the form data we need to submit */
          formData.append('kat', this.kat)
          formData.append('fotoLapor', this.file)
          formData.append('desk', this.desk)
          formData.append('lat', this.lat)
          formData.append('lng', this.lng)
          formData.append('lokasi', this.address)
          formData.append('district', this.district)
          formData.append('status', this.err)

          // Display the key/value pairs
          for (var pair of formData.entries()) {
            console.log(pair[0] + ': ' + pair[1])
          }

          /* Make the request to the POST /single-file URL */
          let result = await axios().post('/report/add', formData)

          // get all admin
          let admins = await axios().get('/user/admin')
          if (admins.data.data.length > 0) {
            // send notif to all admin
            admins.data.data.forEach(async x => {
              await axios().post('/notification/add', {
                recipient_id: x.id,
                report_id: result.data.data.id,
                type: 'Laporan Keluhan',
                desc: `Laporan keluhan baru telah masuk dengan kode ${result.data.data.code}`
              })
            })
          };
          this.openToast('Berhasil menambahkan laporan keluhan')
          this.$f7router.navigate('/home/')
          console.log(result.data)
        }
      } catch (error) {
        console.log('FAILURE!!', error.response)
        this.$f7.dialog.alert(error.response.data.message, 'Terjadi Kesalahan')
      }
    },

    /* Handles a change on the file upload */
    handleFileUpload () {
      this.$refs.actionsOneGroup.close()
      let self = this
      /* Set the local file variable to what the user has selected. */
      this.file = this.$refs.file.files[0]

      /* Initialize a File Reader object */
      var reader = new FileReader()

      /*
          Add an event listener to the reader that when the file
          has been loaded, we flag the show preview as true and set the
          image to be what was read from the reader.
        */
      reader.addEventListener('load', function () {
        this.showPreview = true
        this.imagePreview = reader.result
        // console.log(reader.result);

        this.filename = this.$refs.file.value.replace('C:\\fakepath\\', '')
      }.bind(this), false)

      /* Check to see if the file is not empty. */
      if (this.file) {
        /* Ensure the file is an image file. */
        if (/\.(jpe?g|png|gif)$/i.test(this.file.name)) {
          /*
              Fire the readAsDataURL method which will read the file in and
              upon completion fire a 'load' event which we will listen to and
              display the image in the preview.
            */
          reader.readAsDataURL(this.file)
        }
      }
      this.getLocation(this.file)
    },
    getLocation (file) {
      let self = this
      EXIF.getData(file, function () {
        console.log(this.exifdata)
        self.metadata = this.exifdata

        axios().post('/report/geocode', this.exifdata)
          .then(function (response) {
            console.log(response.data)

            self.err = false
            self.showPreview = true
            self.lat = response.data.data.items[0].position.lat
            self.lng = response.data.data.items[0].position.lng

            self.marker = L.latLng(self.lat, self.lng)
            self.center = L.latLng(self.lat, self.lng)
            self.address = response.data.data.items[0].address.label
            self.district = response.data.data.items[0].address.district

            self.$f7.preloader.hide()
          })
          .catch(function (error) {
            console.log(error.response)
            // self.$f7.dialog.alert(error.response.data.message, 'Terjadi Kesalahan');

            self.err = true
            self.$f7.preloader.hide()
            self.openToast(error.response.data.message)

            self.showPreview = true
            self.marker = L.latLng(0, 0)
            self.address = null
            self.district = null
          })
      })
    },
    removeMarker () {
      // this.markers.splice(index, 1);
    },
    async addMarker (e) {
      try {
        if (this.err == true) {
          this.$f7.preloader.show()
          let response = await axios().post('/report/geocode', {
            lat: e.latlng.lat,
            lng: e.latlng.lng
          })

          console.log(response)

          this.lat = response.data.data.items[0].position.lat
          this.lng = response.data.data.items[0].position.lng

          this.marker = L.latLng(this.lat, this.lng)
          this.center = L.latLng(this.lat, this.lng)
          this.address = response.data.data.items[0].address.label
          this.district = response.data.data.items[0].address.district
          this.$f7.preloader.hide()
          // this.marker = e.latlng;
        } else {
          this.openToast('Foto telah memiliki data lokasi')
        }
      } catch (error) {
        this.address = null
        this.district = null
        console.log(error.message)
        this.$f7.preloader.hide()
        this.openToast('Lokasi tidak ditemukan')
      }
    },
    openToast (text) {
      console.log(text)

      this.toastBottom = this.$f7.toast.create({
        text: text,
        closeTimeout: 3000
      })

      // Open it
      this.toastBottom.open()
    }
  },
  beforeDestroy () {
    document.removeEventListener('backbutton', this.navigateBack)
  }
}
</script>
<style scoped>
.image-upload>input {
  display: none;
}

.preview {
  display: block;
}

.first {
  display: none;
}
.spanku {
  display: block;
}

/* HIDE RADIO */
[type=radio] {
  position: absolute;
  opacity: 0;
  width: 0;
  height: 0;
}

/* IMAGE STYLES */
[type=radio] + i {
  cursor: pointer;
}

/* CHECKED STYLES */
[type=radio]:checked + i {
  color: #007aff;
}
</style>
<style>
.leaflet-popup-close-button {
  display: none;
}
</style>
