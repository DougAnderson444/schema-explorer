<script>
	import { LucidSuggest } from "./en.js";
	import ListItem from "./ListItem.svelte";
	import DATA from "./schema.json";
	import IconLoupe from "./IconLoupe.svelte";
	import Legend from "./Legend.svelte";

	export let limit = 15;

	export let url =
		"https://schema.org/version/latest/schemaorg-current-https.jsonld";

	const suggest = new LucidSuggest();
	suggest.setLimit(limit);

	setUp(DATA);

	async function setUp(data) {
		const start = Date.now();
		// fetch the latest schema data
		try {
			const response = await fetch(url);
			const allData = await response.json();
			data = allData["@graph"];
		} catch (error) {
			console.error(error);
		}

		const rating = 0;
		let id = 0;
		for (const record of data) {
			let title = record["@id"]
				.split(":")[1]
				// insert a space before capitalized words (but not acronyms)
				.replace(/((?<!^)[A-Z](?![A-Z]))(?=\S)/g, " $1")
				// uppercase the first character
				.replace(/^./, (s) => s.toUpperCase());

			for (const property in record) {
				if (
					(typeof record[property] === "string" ||
						record[property] instanceof String) &&
					property !== "@id" && // we already included id above
					property !== "@type" && // we are not including type
					!property.includes("label") // label is the same as the parsed @id
				) {
					title += ` ${record[property]} `; // concat it to the rest
				}
			}
			suggest.addRecords([{ ...record, id, title, rating }]);
			id++;
		}
		const ttc = Date.now() - start;
		console.log("Time to load (ms): ", ttc);
	}

	let input, hits;

	$: input &&
	input.replace(/[`~!@#$%^&*()|+=?;:'",.<>\{\}\[\]\\\/]/gi, "").toLowerCase()
		? (hits = suggest.search(input))
		: null;
</script>

<svelte:head>
	<link
		rel="stylesheet"
		href="https://stackpath.bootstrapcdn.com/bootstrap/4.4.1/css/bootstrap.min.css"
	/>
</svelte:head>
<main>
	<div class="input-group mb-3">
		<input
			bind:value={input}
			class="form-control"
			type="text"
			placeholder="Search Schema.org for 'ATM', or 'Legal Advice'"
		/>
		<div class="input-group-append">
			<span class="input-group-text" id="basic-addon2">
				<IconLoupe />
			</span>
		</div>
	</div>
	{#if hits}
		{#await hits then hits}
			<div class="alert alert-success" role="alert">
				Showing {hits.length < limit ? "" : "the first"}
				<strong>{hits.length}</strong> results.
			</div>
			<div>
				<Legend />
			</div>
		{/await}
	{/if}
	<ul class="list-group" id="search-results">
		{#if hits}
			{#await hits then hits}
				{#each hits as hit}
					<ListItem {hit} bind:input />
				{/each}
			{/await}
		{/if}
	</ul>
</main>

<style>
	main {
		text-align: left;
		padding: 1em;
		max-width: 320px;
		margin: 0 auto;
	}

	@media (min-width: 481px) {
		main {
			max-width: none;
		}
	}
</style>
