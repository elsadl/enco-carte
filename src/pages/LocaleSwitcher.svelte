<script>
  import { createEventDispatcher } from "svelte";
  import { locale } from "svelte-i18n";

  let locales = ["de", "en", "es", "fr", "it"];

  const dispatch = createEventDispatcher();

  function handleLocaleChange(event) {
    event.preventDefault();

    dispatch("locale-changed", event.target.lang);
  }
</script>

<div class="locale-selector">
  <ul>
    {#each locales as lang}
      <li
        {lang}
        class:selected={$locale === lang}
        on:click={handleLocaleChange}
      >
        {lang}
      </li>
    {/each}
  </ul>
</div>

<style>
  ul li {
    cursor: pointer;
    color: var(--primary);
    font-weight: 600;
    display: inline-block;
    padding-bottom: 0.1em;
    text-transform: uppercase;
    font-size: 0.9em;
  }

  ul li:hover {
    background-color: var(--light);
  }

  ul li.selected {
    color: black;
    border-bottom: 2px solid var(--primary);
  }

  ul li + li {
    margin-left: 0.6em;
  }
</style>
