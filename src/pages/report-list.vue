<template>
  <f7-page ptr @ptr:refresh="pullToRefresh">
    <f7-navbar title="Laporan Keluhan" back-link="Back"></f7-navbar>
      <div v-if="items.length == 0" style="padding-top: 300px;">
        <center><span style="font-size: 17px; color: #8e8e93;">Tidak Ada Data Laporan Keluhan.</span></center>
      </div>
      <div v-else>
        <f7-card class="demo-card-header-pic" v-for="(item, index) in items" :key="index">
          <a :href="'/report-detail/' + item.id">
          <f7-card-header
            :id-lapor="item.id"
            class="no-border"
            valign="bottom"
            :style="{ backgroundImage: 'url(' + baseURL + item.image + ')' }"
          ><span><f7-icon md="material:place"></f7-icon>{{item.location}}</span></f7-card-header>
          </a>
          <f7-card-content>
            <p class="date">{{formatTgl(item.created_at)}}</p>
            <p>{{item.description | limitToDisplay(item.description)}}</p>
            <small style="color:red" v-show="item.status == 'Ditolak'">*{{item.reject_notes}}</small>
          </f7-card-content>
          <f7-card-footer>
            <f7-chip v-if="item.status == 0" text="Pending" color="orange"></f7-chip>
            <f7-chip v-else-if="item.status == 1" text="Proses" color="blue"></f7-chip>
            <f7-chip v-else-if="item.status == 2" text="Selesai" color="green"></f7-chip>
            <f7-chip v-else text="Ditolak" color="red"></f7-chip>
            <span>{{item.code}}</span>
          </f7-card-footer>
        </f7-card>
      </div>
  </f7-page>
</template>

<script>
import axios from '../config/axiosConfig'
import date from '../mixins/dateConfig'

export default {
  data () {
    return {
      baseURL: '',
      items: []
    }
  },
  async created () {
    try {
      // Listen to Cordova's backbutton event
      document.addEventListener('backbutton', this.navigateBack, false)
      let baseURL = await axios().request()
      this.baseURL = baseURL.config.baseURL
      this.$f7.preloader.show()
      let result = await axios().get('/report/user')
      this.$f7.preloader.hide()
      console.log(result.data.data)

      this.items = result.data.data
    } catch (error) {
      console.log(error.response)
      this.$f7.dialog.alert(error.response.data.message, 'Terjadi Kesalahan')
    }
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
    async pullToRefresh (event, done) {
      let self = this

      setTimeout(async () => {
        let result = await axios().get('/report/user')
        self.items = result.data.values

        done()
      }, 1000)
    }
  },
  beforeDestroy () {
    document.removeEventListener('backbutton', this.navigateBack)
  },
  filters: {
    limitToDisplay (text) {
      if (text.length < 100) {
        return text
      } else {
        return text.substring(0, 100) + ' ....'
      }
    }
  },
  mixins: [date]
}
</script>
<style scoped>
.demo-card-header-pic .card-header {
  height: 40vw;
  background-size: cover;
  background-position: center;
  color: #fff;
}
.demo-card-header-pic .card-content-padding .date{
  color: #8e8e93;
}
</style>
