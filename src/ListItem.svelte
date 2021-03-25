<script>
    import { onMount } from "svelte";
    // because https://github.com/sveltejs/sapper/issues/904
    onMount(() => {
        document.querySelectorAll("a").forEach((a) => {
            if (!a.hash || !document.querySelectorAll(a.hash).length) return;
            // a.href = window.location.origin + window.location.pathname + a.hash;
        });
    });

    export let hit;
    export let input;

    let links = hit.record["rdfs:comment"].match(/[^([[)]+(?=]])/g);
    let comments = format(hit.record["rdfs:comment"]);
    let domainIncludes = hit.record["schema:domainIncludes"];
    let rangeIncludes = hit.record["schema:rangeIncludes"];
    let subClassOf = hit.record["rdfs:subClassOf"];
    let supersededBy = hit.record["schema:supersededBy"];
    let subPropertyOf = hit.record["rdfs:subPropertyOf"];
    let source = hit.record["schema:source"];

    export const tags = [
        {
            data: domainIncludes,
            style: "card-link badge badge-secondary text-light",
        },
        {
            data: rangeIncludes,
            style: "card-link badge badge-success text-light",
        },
        {
            data: subClassOf,
            style: "card-link badge badge-warning text-dark",
        },
        {
            data: supersededBy,
            style: "card-link badge badge-danger text-light",
        },
        {
            data: subPropertyOf,
            style: "card-link badge badge-info",
        },
    ];
    function formatTitle(title) {
        return (
            title
                // insert a space before capitalized words (but not acronyms)
                .replace(/((?<!^)[A-Z](?![A-Z]))(?=\S)/g, " $1")
                // uppercase the first character
                .replace(/^./, (s) => s.toUpperCase())
        );
    }
    function parseSchema(value) {
        // sometime this is a string
        if (typeof value === "string" || value instanceof String) {
            return value.split(":")[1];
        }

        // sometimes it's an object
        if (value.hasOwnProperty("@id")) return value["@id"].split(":")[1];

        // other sometimes it's an array
        if (value[0].includes(":")) return value[0].split(":")[1];
    }

    function format(text) {
        text = text.replace(/(?:\r\n|\r|\n|\\n)/g, "");

        // skip if no links to see here
        if (!links) return text;

        // insert hyperlinks
        // delimit on links
        // return chunks or links/no links
        const textChunks = text.split(/\[\[\w+\]\]/g);
        return textChunks;
    }
</script>

<!-- {#each hit.chunks as chunk, i}
        {#if chunk.highlight}
            <strong key={i}>{@html format(chunk.text)}</strong>
        {:else}
            <span key={i}>{@html format(chunk.text)}</span>
        {/if}
    {/each} -->

<div
    class="card"
    style="width: max-width: 310px; margin: 0.25em 0; word-wrap: break-word;"
>
    <div class="card-body">
        <h5 class="card-title">{formatTitle(hit.record["rdfs:label"])}</h5>
        <h6 class="card-subtitle mb-2 text-muted">
            {parseSchema(hit.record["@type"])}
        </h6>
        <p class="card-text">
            {#if Array.isArray(comments)}
                {#each comments as comment, i}
                    {@html comment}{#if links[i]}
                        <!-- svelte-ignore a11y-invalid-attribute -->
                        <a
                            href="{window.location.pathname}#top-search-bar"
                            on:click={() => (input = formatTitle(links[i]))}
                            >{formatTitle(links[i])}</a
                        >
                    {/if}
                {/each}
            {:else}
                {@html comments}
            {/if}
        </p>
        {#if links}
            {#each links as link}
                <!-- svelte-ignore a11y-invalid-attribute -->
                <a
                    href="{window.location.pathname}#top-search-bar"
                    class="card-link badge badge-primary text-light"
                    on:click={() => (input = formatTitle(link))}
                    >{formatTitle(link)}</a
                >
            {/each}
        {/if}

        {#each tags as tag}
            {#if tag.data && tag.data.hasOwnProperty("@id")}
                <!-- is an object -->
                {#each [...Object.entries(tag.data)] as [key, value]}
                    <!-- svelte-ignore a11y-invalid-attribute -->
                    <a
                        href="{window.location.pathname}#top-search-bar"
                        class={tag.style}
                        on:click={() =>
                            (input = formatTitle(parseSchema(value)))}
                        >{formatTitle(parseSchema(value))}</a
                    >
                {/each}
            {/if}

            {#if tag.data && tag.data.length > 0}
                <!-- is an array -->
                {#each tag.data as incl}
                    <!-- svelte-ignore a11y-invalid-attribute -->
                    <a
                        href="{window.location.pathname}#top-search-bar"
                        class={tag.style}
                        on:click={() =>
                            (input = formatTitle(parseSchema(incl)))}
                        >{formatTitle(parseSchema(incl))}</a
                    >
                {/each}
            {/if}
        {/each}

        {#if source}
            {#if source && source.hasOwnProperty("@id")}
                <!-- is an object -->
                {#each [...Object.entries(source)] as [key, value]}
                    <!-- svelte-ignore a11y-invalid-attribute -->
                    <br /><a
                        href={value}
                        class="card-link badge badge-light"
                        target="_blank">{value}</a
                    >
                {/each}
            {/if}

            {#if source && source.length > 0}
                <!-- is an array -->
                {#each source as incl}
                    <!-- svelte-ignore a11y-invalid-attribute -->
                    <br /><a
                        href={incl["@id"]}
                        target="_blank"
                        class="card-link badge badge-light">{incl["@id"]}</a
                    >
                {/each}
            {/if}
        {/if}
    </div>
</div>
