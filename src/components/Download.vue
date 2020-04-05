<template>
  <div class="container">
    <h3>Download</h3>
    <slot />
    <p v-if="!budget">Loading budget information...</p>
    <p v-else>Size: {{ budgetSize }}</p>
    <button class="btn btn-primary" @click="downloadBudget">
      Download
    </button>
  </div>
</template>

<script>
  import prettyBytes from 'pretty-bytes';

  export default {
    props: ['budget'],
    computed: {
      budgetJson() {
        return JSON.stringify(this.budget);
      },
      budgetSize() {
        return this.budget ? prettyBytes(this.budgetJson.length) : '<none>';
      },
      downloadFilename() {
        // return `${this.budget.data.name.replace(/ /g, '-')}-ynab-budget-export.json`;
        return `ynab-budget-export.json`;
      }
    },
    methods: {
      downloadBudget() {
        const blob = new Blob([this.budgetJson], { type: 'application/json' });
        const url = window.URL.createObjectURL(blob);
        const link = document.createElement('a');
        link.href = url;
        link.download = this.downloadFilename;
        link.click();
        window.URL.revokeObjectURL(url);
      }
    }
  };
</script>
