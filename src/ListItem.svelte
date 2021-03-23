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
                            href="{window.location.pathname}#"
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
                    href="{window.location.pathname}#"
                    class="card-link badge badge-primary text-light"
                    on:click={() => (input = formatTitle(link))}
                    >{formatTitle(link)}</a
                >
            {/each}
        {/if}

        {#if domainIncludes && domainIncludes.hasOwnProperty("@id")}
            <!-- subclass is an object -->
            {#each [...Object.entries(domainIncludes)] as [key, value]}
                <!-- svelte-ignore a11y-invalid-attribute -->
                <a
                    href="{window.location.pathname}#"
                    class="card-link badge badge-secondary text-light"
                    on:click={() => (input = formatTitle(parseSchema(value)))}
                    >{formatTitle(parseSchema(value))}</a
                >
            {/each}
        {/if}

        {#if domainIncludes && domainIncludes.length > 0}
            {#each domainIncludes as incl}
                <!-- svelte-ignore a11y-invalid-attribute -->
                <a
                    href="{window.location.pathname}#"
                    class="card-link badge badge-secondary text-light"
                    on:click={() => (input = formatTitle(parseSchema(incl)))}
                    >{formatTitle(parseSchema(incl))}</a
                >
            {/each}
        {/if}

        {#if rangeIncludes && rangeIncludes.hasOwnProperty("@id")}
            <!-- subclass is an object -->
            {#each [...Object.entries(rangeIncludes)] as [key, value]}
                <!-- svelte-ignore a11y-invalid-attribute -->
                <a
                    href="{window.location.pathname}#"
                    class="card-link badge badge-success text-light"
                    on:click={() => (input = formatTitle(parseSchema(value)))}
                    >{formatTitle(parseSchema(value))}</a
                >
            {/each}
        {/if}

        {#if rangeIncludes && rangeIncludes.length > 0}
            {#each rangeIncludes as incl}
                <!-- svelte-ignore a11y-invalid-attribute -->
                <a
                    href="{window.location.pathname}#"
                    class="card-link badge badge-success text-light"
                    on:click={() => (input = formatTitle(parseSchema(incl)))}
                    >{formatTitle(parseSchema(incl))}</a
                >
            {/each}
        {/if}

        {#if subClassOf && subClassOf.hasOwnProperty("@id")}
            <!-- subclass is an object -->
            {#each [...Object.entries(subClassOf)] as [key, value]}
                <!-- svelte-ignore a11y-invalid-attribute -->
                <a
                    href="{window.location.pathname}#"
                    class="card-link badge badge-warning text-dark"
                    on:click={() => (input = formatTitle(parseSchema(value)))}
                    >{formatTitle(parseSchema(value))}</a
                >
            {/each}
        {/if}

        {#if subClassOf && subClassOf.length > 0}
            <!-- subclass is an array of objects -->
            {#each subClassOf as incl}
                <!-- svelte-ignore a11y-invalid-attribute -->
                <a
                    href="{window.location.pathname}#"
                    class="card-link badge badge-warning text-dark"
                    on:click={() => (input = formatTitle(parseSchema(incl)))}
                    >{formatTitle(parseSchema(incl))}</a
                >
            {/each}
        {/if}
    </div>
</div>
