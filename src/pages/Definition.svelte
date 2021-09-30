<script>
  import { dictionary, locale, _ } from "svelte-i18n";
  import { get } from "svelte/store";

  export let name;
  export let classe;
  export let lowercase;

  function checkTranslation(string) {
    if (string in get(dictionary)[$locale]) {
      return true;
    } else if (
      string.replace(" ", "_").toLowerCase() in get(dictionary)[$locale]
    ) {
      return true;
    } else {
      return false;
    }
  }

  function getTranslation(string) {
    if (string in get(dictionary)[$locale]) {
      return get(dictionary)[$locale][string];
    } else if (
      string.replace(" ", "_").toLowerCase() in get(dictionary)[$locale]
    ) {
      return capitalizeFirstLetter(
        get(dictionary)[$locale][string.replace(" ", "_").toLowerCase()]
      );
    } else {
      return string;
    }
  }
</script>

<!-- svelte-ignore a11y-missing-attribute -->
<a class="definition">
  {#if checkTranslation(name + "_def")}
    <span class="popup {classe}">
      <span>{getTranslation(name + "_def")}</span>
    </span>
  {/if}
  {#if lowercase}
    <span class="name {classe}">{getTranslation(name).toLowerCase()}</span>
  {:else}
    <span class="name {classe}">{getTranslation(name)}</span>
  {/if}
</a>

<style>
  .name {
    font-weight: 600;
  }

  .popup + .cat1 {
    border-bottom: 2px dotted var(--primary);
  }

  .popup + .cat2 {
    border-bottom: 2px dotted var(--secondary);
  }

  .popup {
    pointer-events: none;
    display: none;
    z-index: 999;
    color: #000;
    font-size: 0.75em;
    position: absolute;
    width: 240px;
    top: 2.6em;
    padding: 1em;
    background-color: #fff;
  }

  .popup::before {
    content: "";
    display: block;
    position: absolute;
    left: 15px;
    bottom: 100%;
    width: 0;
    height: 0;
    border: 8px solid transparent;
  }

  .popup::after {
    content: "";
    display: block;
    position: absolute;
    left: 16px;
    bottom: 100%;
    width: 0;
    height: 0;
    border: 7px solid transparent;
    border-bottom-color: white;
  }

  .popup.cat1 {
    border: 1px solid var(--primary);
  }

  .popup.cat2 {
    border: 1px solid var(--secondary);
  }

  .popup.cat1::before {
    border-bottom-color: var(--primary);
  }

  .popup.cat2::before {
    border-bottom-color: var(--secondary);
  }

  .popup-title {
    font-weight: bold;
    display: inline-block;
    padding-bottom: 0.4em;
  }

  a {
    display: inline-block;
    position: relative;
  }

  a:hover {
    opacity: 1;
  }

  a:hover .popup {
    display: block;
  }

  @media (max-width: 600px) {
    a {
      position: unset;
    }

    .graph-container {
      position: relative;
    }

    .popup {
      width: 100%;
      top: 3em;
      left: 0;
    }

    .popup.cat2::before {
      left: unset;
      right: 49px;
    }

    .popup.cat2::after {
      left: unset;
      right: 50px;
    }
  }
</style>
