<script>
	import axios from "axios";
	import { slide } from "svelte/transition";
	import Transaction from "./components/Transaction.svelte";
	import SummaryCard from "./components/SummaryCard.svelte";
	import Loading from "./components/Loading.svelte";
	import { onMount } from "svelte";
	import {
		transactions,
		sortedTransactions,
		balance,
		income,
		expenses,
	} from "./components/stores";

	let input = 0;
	let perihal;
	let typeOfTransaction = "+";
	let loading = false;

	$: disabled = !input;

	onMount(async () => {
		loading = true;
		const { data } = await axios.get("/api/transactions");
		$transactions = data;
		loading = false;
	});

	async function addTransaction() {
		const transaction = {
			date: new Date().getTime(),
			value: typeOfTransaction === "+" ? input : input * -1,
			perihal,
		};
		const response = await axios.post("/api/transactions", transaction);
		$transactions = [response.data, ...$transactions];
		input = 0;
	}

	async function removeTransaction(id) {
		const response = await axios.delete("/api/transactions/" + id);
		if (response.data.id === id) {
			$transactions = $transactions.filter((t) => t._id !== id);
		}
	}
</script>

<style>
	.app {
		margin: 40px auto;
		max-width: 500px;
	}
</style>

<div class="app container">
	<div class="field has-addons">
		<p class="control">
			<span class="select">
				<select bind:value={typeOfTransaction}>
					<option value="+">+</option>
					<option>-</option>
				</select>
			</span>
		</p>
		<p class="control is-expanded">
			<input
				class="input"
				type="text"
				bind:value={perihal}
				placeholder="Perihalnya" />
		</p>
		<p class="control is-expanded">
			<input
				class="input"
				type="number"
				bind:value={input}
				placeholder="Jumlah Duitnya" />
		</p>
		<p class="control">
			<button class="button" on:click={addTransaction} {disabled}>
				Simpan
			</button>
		</p>
	</div>

	{#if loading}
		<Loading />
	{/if}

	{#if $transactions.length > 0}
		<SummaryCard mode="saldo'ta" value={$balance} />

		<div class="columns">
			<div class="column">
				<SummaryCard mode="pemasukan" value={$income} />
			</div>

			<div class="column">
				<SummaryCard mode="pengeluaran" value={$expenses} />
			</div>
		</div>
		<hr />
	{:else if !loading}
		<div
			class="notification has-text-centered"
			transition:slide={{ duration: 1000 }}>
			Masuk-kan transaksi pertamamu Cika'
		</div>
	{/if}
	{#each $sortedTransactions as transaction (transaction._id)}
		<Transaction {transaction} {removeTransaction} />
	{/each}
</div>
