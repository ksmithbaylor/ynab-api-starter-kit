<template>
  <div class="budgets container">
    <h3>Budget</h3>
    <p class="lead">Please select which budget you'd like to download.</p>
    <p v-if="loadingBudgets">Loading budget list...</p>
    <ul v-else>
      <li v-for="budget in budgets">
        <a href="#" @click="selectBudget(budget.id, $event)">
          {{ budget.name }}
        </a>
        - last modified on {{ formatDate(budget.last_modified_on) }}
      </li>
    </ul>
  </div>
</template>

<script>
  export default {
    props: ['budgets', 'selectBudget', 'loadingBudgets'],
    methods: {
      formatDate(date) {
        return new Date(date).toString();
      }
    }
  };
</script>
