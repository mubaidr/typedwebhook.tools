<script lang="ts">
  import Highlight from 'svelte-highlight';
  import { json as jsonLanguage } from 'svelte-highlight/src/languages';
  import { githubDark } from 'svelte-highlight/src/styles';
  import Tabs from '$lib/Tabs.svelte';
  import Types from './Types.svelte';
  import url from '$lib/url';

  export let preview = false;
  export let method = 'POST';
  export let body = '';
  export let size = -1;
  export let headers = {};
  export let time = '';

  export let selected = [];

  // Which view we have enabled, based off of the hash.

  // Extract all headers from
  $: headerKeys = Object.keys(headers || {})
    .sort((a, b) => a.localeCompare(b))
    .filter((key) => key.indexOf('cf-') !== 0);

  $: [json, isJSON] = (() => {
    try {
      return [JSON.stringify(JSON.parse(body), null, '  '), true];
    } catch (e) {
      return [undefined, false];
    }
  })(body);

  if (size === -1) {
    $: size = body.length;
  }
</script>

<svelte:head>
  {@html githubDark}
</svelte:head>

<div class="request-data" class:preview>
  <h2>Request data</h2>
  {#if preview}
    <p>Waiting for your first request...</p>
  {:else}
    {#if selected.length == 1}
      <p>Request received from <code>{(headers || {})['x-real-ip']}</code></p>
    {:else}
      <p>{selected.length} requests selected. Showing a single type which allows all requests.</p>
    {/if}
  {/if}

  {#if selected.length <= 1}
    <div class="header-table">
      <div class="th">Header</div>
      <div class="th">Value</div>
      {#each headerKeys as header}
        <div class="header-name">{header}</div>
        <div>{(headers || {})[header]}</div>
      {/each}
    </div>
  {/if}

  <div class="body">


    {#if selected.length <= 1}
      <Tabs
        current={$url?.hash || '#body'}
        tabs={[
          { label: 'Body', href: '#body', onClick: () => {} },
          { label: 'JSON', href: '#json', onClick: () => {} },
          { label: 'Types', href: '#types', onClick: () => {} }
        ]}
      />
    {:else}
      <Tabs
        current={$url?.hash || '#types'}
        tabs={[
          { label: 'Types', href: '#types', onClick: () => {} }
        ]}
      />
    {/if}

    <div class="body-content">
      {#if ($url?.hash || '#body') === '#body' && selected.length <= 1}
        <code class="pre">{body || 'No data'}</code>
      {/if}

      {#if $url?.hash === '#json'}
        {#if !isJSON}
          <p class="no-json">This request is not valid JSON 😢</p>
        {:else}
          <div class="json">
            <Highlight language={jsonLanguage} code={json} />
          </div>
        {/if}
      {/if}

      {#if $url?.hash === '#types' || selected.length > 1}
        {#if !isJSON}
          <p class="no-json">
            This request is not valid JSON, so we can't make types for this body 😢
          </p>
        {:else}
          <div class="json">
            <Types {body} multiple={selected.map(i => i.body)} />
          </div>
        {/if}
      {/if}
    </div>
  </div>
</div>

<style>
  .preview {
    opacity: 0.8;
  }

  .request-data {
    padding: 1.5rem 5rem;
    background: var(--main-bg);
  }

  .header-table {
    display: grid;
    grid-template-columns: auto 3fr;
    grid-template-rows: auto;
    margin: 2rem 0 2rem;
    font-family: var(--font-mono);
    position: relative;
  }

  .header-table:before,
  .body:before {
    font-family: var(--font-mono);
    display: block;
    position: absolute;
    left: -67px;
    top: 8px;
    font-size: 0.65rem;
    text-transform: uppercase;
  }

  .body {
    position: relative;
    margin: 2rem 0 0;
  }
  .body-content {
    margin-top: 1rem;
  }
  .header-table:before {
    content: 'Headers';
  }

  .body:before {
    content: 'Body';
    top: 10px;
  }

  .header-table div {
    padding: 0.25rem 0;
    border-bottom: 1px solid var(--border-color);
    display: flex;
    align-items: center;
  }

  .header-name {
    /* We use this instead of grid gap so that the border extends across the row */
    padding-right: 40px !important;
    font-family: var(--font);
    font-size: 0.7rem;
    opacity: 0.8;
  }

  .header-name + div {
    font-size: 0.75rem;
  }

  .header-table .th {
    border-bottom: 1px solid #eee;
    margin-bottom: 0.25rem;
    opacity: 0.5;
  }

  .body code {
    display: block;
  }

  .json {
    font-family: var(--font-mono);
    background-color: var(--code-bg);
    padding: 1rem;
    margin: 0 -0.5rem; /* match pre */
    border-radius: var(--border-radius);
  }

  .th:first-of-type {
    min-width: 120px;
  }

  .no-json {
    /* margin: 2rem 0; */
    text-align: center;
  }
</style>
