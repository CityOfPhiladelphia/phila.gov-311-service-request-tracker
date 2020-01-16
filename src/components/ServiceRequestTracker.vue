
<template>
  <div class="widget">
    <p>You can use this tool to track the status of your 311 service request.</p>

    <div class="search">
      Type in your service request number.
      <input
        id="search-bar"
        v-model="serviceRequestNumber"
        class="search-field"
        type="text"
        placeholder="Search by document name or meeting number"
      />
      <input 
        ref="request-search-bar" 
        type="submit" 
        class="search-submit" 
        value="Search" 
        @click="requestData"
      />
      <button
        v-if="serviceRequestNumber.length > 0"
        class="clear-search-btn"
        @click="clearSearchBar"
      >
        <i class="fas fa-times" />
      </button>
    </div>

    <div v-if="serviceRequestData" class="service-request-data">
       <table role="grid" class="responsive">
        <thead>
        
        </thead>
        <tbody>
          <tr>
          <th scope="row">Type</th>
          <td>{{ serviceRequestData.service_name }}</td>
          </tr>
           <tr>
          <th scope="row">Status</th>
          <td>{{ serviceRequestData.status }}</td>
           </tr>
           <tr>
          <th scope="row">Department</th>

          
           <td>{{ serviceRequestData.agency_responsible }}</td>
           </tr>
           <tr>
          <th scope="row">Resolution</th>
          <td>{{ serviceRequestData.status_notes }}</td> 
          </tr>
        
      </tbody>
      </table>
    </div>
  </div>
</template>
<script>
/* eslint-disable no-console */
import axios from "axios";
const endpoint = "https://api.phila.gov/open311/v2/requests/";

export default {
  name: "ServiceRequestTracker",
  data: function() {
    return {
      serviceRequestNumber: "13075315",
      serviceRequestData: null,
      requested: false,
      wrongNumber: false,
      loading: false
    };
  },
  watch: {
    serviceRequestData(objArr) {
      this.update(objArr);
    }
  },
  methods: {
    update() {
      console.log("updated!");
    },

    clearSearchBar() {
      this.serviceRequestNumber = "";
      this.serviceRequestData = null;
    },

    requestData() {
      this.loading = true;
      let reqUrl = endpoint + this.serviceRequestNumber + ".json";

      axios
        .get(reqUrl)
        .then(response => {
          this.serviceRequestData = response.data[0];
          this.loading = false;
        })
        .catch(e => {
          this.wrongNumber = true;
          window.console.log(e);
        });
    }
  },
  created: function() {
    // this.requestData();
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss">
.widget {
  width: 50rem;
  margin: 0 auto;
}

.search-wrapper {
  padding-bottom: 1rem;

  .clear-search-btn {
    position: absolute;
    top: 16px;
    right: 70px;
    padding: 0;
    font-size: 20px;
    background-color: #fff;
    opacity: 0.8;
    z-index: 999;
    cursor: pointer;
    color: rgba(60, 60, 60, 0.5);
  }

  .clear-button-wrap {
    height: 50%;
    float: right;

    button {
      margin-left: 10px;
    }

    .clear-button {
      background-color: transparent;
      text-decoration: underline;
      color: black;
      //color: white;
      &:hover {
        background-color: transparent;
        color: grey;
      }
    }
  }
}
</style>
