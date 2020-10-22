<script>
  import { slide } from "svelte/transition";
  export let transaction;
  export let removeTransaction;
</script>

<style>
  .date {
    float: right;
  }
</style>

<div
  class="notification is-light {transaction.value > 0 ? 'is-success' : 'is-danger'} "
  transition:slide={{ duration: 800 }}>
  {transaction.value}

  <span class="is-size-7 has-text-grey date">
    {transaction.perihal}
    {new Date(transaction.date).toLocaleString()}
  </span>
  <button class="delete" on:click={() => removeTransaction(transaction._id)} />
</div>
