<template>
  <div class="hello">
    <p>You can use this tool to track the status of your 311 service request.</p>

    <input type="text" />



  </div>
  

</template>
<script>

import axios from "axios";
const endpoint = 'http://api.phila.gov/open311/v2/requests/';

export default {
  name: 'ServiceRequestTracker',
  data: function() {
    return {
      serviceRequestNumber: '13075315',
      serviceRequestData: [{}],
      requested: false,
      wrongNumber: false,
      loading: false,
    }
  },
  watch: {
    serviceRequestData(objArr) {
      this.update(objArr);
    }
  },
  methods: {

    update() {

    },

    requestData() {
      this.loading = true;
      let reqUrl = endpoint + this.serviceRequestNumber + ".json";

      axios.get(reqUrl)
      .then(response => {
        this.serviceRequestData = response.data;
        this.loading = false;
      }).catch(e => {
        this.wrongNumber = true;
        window.console.log(e);
      })

    },
    

  },
  created: function() {
    this.requestData();
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
