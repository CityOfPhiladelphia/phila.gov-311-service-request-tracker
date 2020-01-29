
<template>
  <div class="widget">
    <label for="service-search">Enter your 8-digit service request number.</label>
    <div class="search">
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
      <input
        ref="request-search-bar"
        type="submit"
        class="search-submit"
        value="Search"
        @click="requestData()"
      />
      <button v-if="id" class="clear-search-btn" @click="clearSearchBar()">
        <i class="fas fa-times" />
      </button>
    </div>
    <div v-if="failure && !loading">
      <strong class="error-message">The service request number you entered is invalid.</strong>
    </div>
    <div v-if="loading" class="mtm center">
      <i class="fas fa-spinner fa-spin fa-3x" />
    </div>
    <div v-if="serviceRequestData && !failure && !loading" id="service-request-data">
      <div class="row">
        <div class="columns">
          <section class="tertiary-content">
            <h3 class="service-id">Service request #{{ serviceRequestData.service_request_id }}</h3>
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
      serviceRequestData: null,
      failure: false,
      loading: false,
      routerQuery: {}
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
      this.id = "";
    },

    requestData() {
      this.loading = true;
      let reqUrl = endpoint + this.id + ".json";

      axios
        .get(reqUrl)
        .then(response => {
          this.serviceRequestData = response.data[0];
          this.loading = false;
          this.failure = false;
        })
        .catch(e => {
          this.failure = true;
          this.loading = false;
          this.serviceRequestData = null;
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
.widget {
  max-width: 50rem;
  margin: 0 auto;
}

.search {
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
}

.small-hr {
  margin: 17px 10px;
}

.service-id {
  margin-top: 0px !important;
}

.error-message {
  color: #cc3000;
}

</style>