
<template>
  <div class="row">
    <div class="vue-search">
      <input
        name="service-search"
        id="search-bar"
        v-model.number="id"
        class="search-field"
        pattern="\d*"
        type="text"
        maxlength=8
        placeholder="For example: 10808044"
        @keyup.enter="requestData()"
      />
      <label for="service-search">Enter your 8-digit service request number.</label>
      <button v-if="id" class="clear-search-btn" @click="clearSearchBar()">
        <i class="fas fa-times" />
      </button>
      <button
        class="search-submit"
        @click="requestData()"
      >
        <i class="fa-solid fa-magnifying-glass" />
      </button>
      <div v-if="!serviceRequestData && !failure">
      You can <a href="https://admin.phila.gov/services/property-lots-housing/submit-a-service-request-with-311/">report a variety of problems to 311</a>, including:
      <ul>
        <li>Potholes and street damage.</li>
        <li>Abandoned automobiles.</li>
        <li>Graffiti.</li>
        <li>Illegal dumping.</li>
        <li>Street light outages.</li>
      </ul>
      To track an existing service request, enter your service request number in the tool above.
    </div>
    </div>
    <div v-if="showSearchInput" class="search-input">
      <button class="clear-label" @click="clearSearchBar()">Clear</button>
      <div class="service-request-label">Service Request</div>
      <div class="service-request">#{{ displayID }}</div>
    </div>
    <div v-if="failure && !loading && showSearchInput">
      <strong class="error-message">This service request number was not found. If you marked your initial request as private, you won't be able to track it using this tool. Call 311 for an update on private tickets.</strong>
    </div>
    <div v-if="loading" class="mtm center">
      <i class="fas fa-spinner fa-spin fa-3x" />
    </div>
    <div v-if="serviceRequestData && !failure && !loading" id="service-request-data">
      <div class="row">
        <div class="columns">
          <section class="tertiary-content">
            <div class="mlm">
              <strong>Status</strong>
              <p>{{ serviceRequestData.status | capitalize }}</p>
            </div>
            <hr class="small-hr" />
             <div class="mlm">
              <strong>Date requested</strong>
              <p>{{ serviceRequestData.requested_datetime | dateDisplay }}</p>
            </div>
            <hr class="small-hr" />
            <div v-if="serviceRequestData.address" class="mlm">
              <strong>Address</strong>
              <p>{{ serviceRequestData.address }}, {{ serviceRequestData.zipcode }}</p>
            </div>
            <hr class="small-hr" v-if="serviceRequestData.address" />   
            <div class="mlm">
              <strong>Type</strong>
              <p>{{ serviceRequestData.service_name }}</p>
            </div>
            <hr class="small-hr" />
            <div class="mlm">
              <strong>Department</strong>
              <p>{{ serviceRequestData.agency_responsible }}</p>
            </div>
            <hr v-if="serviceRequestData.status_notes" class="small-hr" />
            <div v-if="serviceRequestData.status_notes" class="mlm">
              <strong>Resolution</strong>
              <p>{{ serviceRequestData.status_notes }}</p>
            </div>
          </section>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import Vue from "vue";
import axios from "axios";
import moment from "moment";
Vue.use(moment);

const endpoint = "https://api.phila.gov/open311/v2/requests/";

export default {
  name: "ServiceRequestTracker",
  data: function() {
    return {
      id: "",
      displayID: "",
      serviceRequestData: null,
      failure: false,
      loading: false,
      routerQuery: {},
      showSearchInput: false 
    };
  },

  filters: {
    capitalize: function(value) {
      if (!value) return "";
      value = value.toString();
      return value.charAt(0).toUpperCase() + value.slice(1);
    },

    dateDisplay: function(val) {
      return moment(val).format("MMMM D, YYYY");
    }
  },
  watch: {
    routerQuery: {
      handler: function() {
        this.updateRouter();
      },
      deep: true
    },

    serviceRequestData() {
      if (this.id) {
        this.updateRouterQuery("id", this.id);
      } else {
        this.updateRouterQuery("id", null);
      }
    },

    id(val) {
      if (val.length == 8) {
        this.requestData();
      }
            this.id = val.replace(/[^0-9]+/g, ''); 
    }
  },
  methods: {
    updateRouterQuery: function(key, value) {
      if (!value) {
        Vue.delete(this.routerQuery, key);
      } else {
        Vue.set(this.routerQuery, key, value);
      }
    },

    updateRouter: function() {
      if (this.routerQuery.id == this.$route.query.id) {
        return;
      }
      this.$router
        .push({
          name: "main",
          query: this.routerQuery
        })
        .catch(e => {
          window.console.log(e);
        });
    },

    init: function() {
      if (Object.keys(this.$route.query).length !== 0) {
        for (let routerKey in this.$route.query) {
          Vue.set(this, routerKey, this.$route.query[routerKey]);
        }
      }
    },

    clearSearchBar() {
      this.id = null;
      this.showSearchInput = false; 
      this.serviceRequestData = null;
      this.failure = false;
    },

    requestData() {
      this.loading = true;
      this.showSearchInput = true; 
      let reqUrl = endpoint + this.id + ".json";

      axios
        .get(reqUrl)
        .then(response => {
          this.serviceRequestData = response.data[0];
          this.loading = false;
          this.failure = false;
          this.displayID = this.id;
        })
        .catch(e => {
          this.failure = true;
          this.loading = false;
          this.serviceRequestData = null;
          this.displayID = this.id;
          window.console.log(e);
        });
    }
  },
  mounted: function() {
    this.init();
  }
};
</script>

<style lang="scss">

.search-input{
      padding: 24px 0 24px 0;
      width: 100%;
      background-color: #daedfe;
      margin-bottom: 24px;
    }
  .service-request-label {
      padding-left: 24px;
      font-size: 16px;
    }
  .service-request {
      padding-left: 24px;
      font-size: 20px;
      font-weight: 200;
    }
  .clear-label{
      all: unset;
      padding: 0 24px 0 0;
      font-weight: 700;
      text-decoration: underline;
      color: #0f4d90;
      float: right;
      text-align: right;
      cursor: pointer;
    }
    .clear-label:hover{
      background-color: transparent;
      color: #0f4d90;
    }
    .clear-label:focus{
      background-color: transparent;
      border: 2px solid #0f4d90;
      color: #0f4d90;
    }

.vue-search {
  padding-bottom: 1rem;
  width: 100%;
  position: relative;

  .search-field{
    margin: 0;
    min-height: 3.8rem;
    border: 2px solid #0f4d90;
    background: white;
  }

  .search-submit{ 
    padding: 0.4rem;
    font-size: 2rem;
    font-weight: 400;
    background: #0f4d90;
    color: white;
    position: absolute;
    top: 0;
    right: 0;
    width: 3.8rem;
    height: 3.8rem;
    cursor: pointer;
  }

  .fa-magnifying-glass{
    font-weight: normal;
  }

  label{
    margin: 7px 0 20px 8px;
  }

  .clear-search-btn {
    position: absolute;
    top: 16px;
    right: 70px;
    padding: 0;
    font-size: 20px;
    background-color: #fff;
    opacity: 0.8;
    cursor: pointer;
    color: rgba(60, 60, 60, 0.5);
      &:hover {
      background: transparent;
      color: black;
    }
  }
}

.small-hr {
  margin: 17px 10px;
}

.error-message {
  font-weight: 200;
  font-size: 20px; 
}

</style>