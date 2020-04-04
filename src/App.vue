<template>
  <div id="app">
    <Nav />
    <div class="container">
      <!-- Display an error if we got one -->
      <div v-if="error">
        <h3>Oops!</h3>
        <p class="lead">Something went wrong:</p>
        <p>{{ error }}</p>
      </div>

      <!-- Otherwise show our app contents -->
      <div v-else>
        <!-- If we dont have a token ask the user to authorize with YNAB -->
        <form v-if="!ynab.token">
          <div class="form-group">
            <h3>Welcome!</h3>
            <p class="lead">To get started, please authorize with YNAB.</p>
            <button @click="authorizeWithYNAB" class="btn btn-primary">
              Authorize &gt;
            </button>
          </div>
        </form>

        <!-- Otherwise if we have a token, show the budget select -->
        <Budgets
          v-else-if="!budgetId"
          :budgets="budgets"
          :selectBudget="selectBudget"
          :loadingBudgets="loadingBudgets"
        />

        <!-- If a budget has been selected, display transactions from that budget -->
        <div v-else>
          <Download :budget="budget">
            <a
              href="#"
              @click="clearBudget"
              style="display: block; margin-bottom: 1.5em"
            >
              &lt; Select a different budget
            </a>
          </Download>
        </div>
      </div>

      <Footer />
    </div>
  </div>
</template>

<script>
  // Hooray! Here comes YNAB!
  import * as ynab from 'ynab';

  // Import our config for YNAB
  import config from './config.json';

  // Import Our Components to Compose Our App
  import Nav from './components/Nav.vue';
  import Footer from './components/Footer.vue';
  import Budgets from './components/Budgets.vue';
  import Download from './components/Download.vue';

  export default {
    // The data to feed our templates
    data() {
      return {
        ynab: {
          clientId: config.clientId,
          redirectUri: config.redirectUri,
          token: null,
          api: null
        },
        loadingBudgets: false,
        error: null,
        budgetId: null,
        budgets: [],
        budget: null
      };
    },

    // When this component is created, check whether we need to get a token,
    // budgets or display the transactions
    created() {
      this.ynab.token = this.findYNABToken();
      if (this.ynab.token) {
        this.api = new ynab.api(this.ynab.token);
        if (!this.budgetId) {
          this.getBudgets();
        } else {
          this.selectBudget(this.budgetId);
        }
      }
    },

    methods: {
      // This uses the YNAB API to get a list of budgets
      getBudgets() {
        this.loadingBudgets = true;
        this.error = null;
        this.api.budgets
          .getBudgets()
          .then(res => {
            this.budgets = res.data.budgets;
            this.loadingBudgets = false;
          })
          .catch(err => {
            this.loadingBudgets = false;
            if (err && err.error && err.error.detail) {
              this.error = err.error.detail;
            } else {
              this.error = err.message;
            }
          })
      },

      // This selects a budget and gets all the transactions in that budget
      selectBudget(id) {
        this.error = null;
        this.budgetId = id;
        this.api.budgets.getBudgetById(id).then(res => {
          this.budget = res.data.budget;
        });
      },

      clearBudget() {
        this.budgetId = null;
        this.loadingBudgets = false;
        this.error = null;
      },

      // This builds a URI to get an access token from YNAB
      // https://api.youneedabudget.com/#outh-applications
      authorizeWithYNAB(e) {
        e.preventDefault();
        const uri = `https://app.youneedabudget.com/oauth/authorize?client_id=${this.ynab.clientId}&redirect_uri=${this.ynab.redirectUri}&response_type=token`;
        location.replace(uri);
      },

      // Method to find a YNAB token
      // First it looks in the location.hash and then sessionStorage
      findYNABToken() {
        let token = null;
        const search = window.location.hash
          .substring(1)
          .replace(/&/g, '","')
          .replace(/=/g, '":"');
        if (search && search !== '') {
          // Try to get access_token from the hash returned by OAuth
          const params = JSON.parse('{"' + search + '"}', function(key, value) {
            return key === '' ? value : decodeURIComponent(value);
          });
          token = params.access_token;
          sessionStorage.setItem('ynab_access_token', token);
          window.location.hash = '';
        } else {
          // Otherwise try sessionStorage
          token = sessionStorage.getItem('ynab_access_token');
        }
        return token;
      },
    },

    // Specify which components we want to make available to our templates
    components: {
      Nav,
      Footer,
      Budgets,
      Download
    }
  };
</script>
