<script>
  import { capitalize, capitalizeFirstLetter } from "../../Utils.svelte";
  import { json } from "d3-fetch";

  import { dictionary, locale, _ } from "svelte-i18n";
  import { get } from "svelte/store";
  export let country;

  let companiesDict;

  json("../data/dict_entreprises.json").then((json) => {
    companiesDict = json;
  });

  function getTranslation(string) {
    if (typeof string != "string") {
      return string;
    }
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

<div class="tooltip">
  <div class="tooltip-header">
    <p class="tooltip-title">
      {capitalize(country["name_" + $locale].replace("_", " "))}
    </p>
    <div class="tooltip-close" />
  </div>
  <div class="tooltip-body">
    <div class="header-grid grid">
      <p />
      <p class="col-label label">{$_("hospitals")}</p>
      <p class="col-label label">{$_("nursing_homes")}</p>
    </div>
    <div class="beds-grid grid">
      <p class="row-label label">{$_("total_beds")}</p>
      <p>
        {@html country.hospitals.total_beds
          ? country.hospitals.total_beds.toLocaleString("de-DE")
          : "—"}
      </p>
      <p>
        {@html country.nursing_homes.total_beds
          ? country.nursing_homes.total_beds.toLocaleString("de-DE")
          : "—"}
      </p>
      <p class="row-label label">{$_("total_beds_per_inhabitants")}</p>
      <p>
        {@html country.hospitals.beds_per_inhabitants
          ? country.hospitals.beds_per_inhabitants.toLocaleString("de-DE") + " ‰"
          : "—"}
      </p>
      <p>
        {@html country.nursing_homes.beds_per_inhabitants
          ? country.nursing_homes.beds_per_inhabitants.toLocaleString("de-DE") + " ‰"
          : "—"}
      </p>
      <p class="row-label label bolder">{$_("%_for_profit_beds")}</p>
      <p class="bolder">
        {#if country.id == "BE"}
          {#if country.hospitals.for_profit_beds}
            {#each country.hospitals.for_profit_beds.split("-") as el}
              {el.trim().substring(0, 2)} %
              <span>({getTranslation(el.trim().match(/\((.*)\)/)[1])})</span>
              <br />
            {/each}
          {:else}
            —
          {/if}
        {:else}
          {@html country.hospitals.for_profit_beds
            ? country.hospitals.for_profit_beds.toLocaleString("de-DE") + " %"
            : "—"}
        {/if}
      </p>
      <p class="bolder">
        {#if country.id == "BE"}
          {#if country.nursing_homes.for_profit_beds}
            {#each country.nursing_homes.for_profit_beds.split("-") as el}
              {el.trim().substring(0, 2)} %
              <span>({getTranslation(el.trim().match(/\((.*)\)/)[1])})</span>
              <br />
            {/each}
          {:else}
            —
          {/if}
        {:else}
          {@html country.nursing_homes.for_profit_beds
            ? country.nursing_homes.for_profit_beds.toLocaleString("de-DE") + " %"
            : "—"}
        {/if}
      </p>
    </div>
    <div class="companies-grid grid">
      <p class="row-label label">{$_("companies")}</p>
      {#if country.hospitals.companies.length == 0}
        <p>—</p>
      {:else if companiesDict}
        <ul>
          {#each country.hospitals.companies as company}
            <li>
              <a class="to-company" href={"/company/" + company}
                >{companiesDict[company]}</a
              >
            </li>
          {/each}
        </ul>
      {/if}
      {#if country.nursing_homes.companies.length == 0}
        <p>—</p>
      {:else if companiesDict}
        <ul>
          {#each country.nursing_homes.companies as company}
            <li>
              <a class="to-company" href={"/company/" + company}
                >{companiesDict[company]}</a
              >
            </li>
          {/each}
        </ul>
      {/if}
    </div>
  </div>
</div>

<style>
  .tooltip {
    background-color: #fff;
    padding: 12px 14px;
    border: 1px solid #ccc;
    border-radius: 6px;
    z-index: 2;
  }

  .tooltip-header {
    padding-bottom: 0.3em;
    margin-bottom: 0.6em;
    border-bottom: 1px solid #ccc;
    display: flex;
    justify-content: space-between;
    align-items: baseline;
  }

  .tooltip-title {
    font-size: 1.8em;
    font-weight: 800;
  }

  .tooltip-close {
    position: relative;
    width: 1em;
    height: 1em;
    cursor: pointer;
    transition: opacity 200ms;
    display: none;
  }

  .tooltip-close:hover {
    opacity: 0.4;
  }

  .tooltip-close:before,
  .tooltip-close:after {
    position: absolute;
    left: 0.5em;
    content: " ";
    height: 1em;
    width: 2px;
    background-color: var(--primary);
  }

  .tooltip-close:before {
    transform: rotate(45deg);
  }

  .tooltip-close:after {
    transform: rotate(-45deg);
  }

  .grid {
    display: grid;
    grid-template-columns: minmax(110px, 1fr) 1fr 1fr;
    grid-row-gap: 0.6em;
    grid-column-gap: 0.8em;
    padding: 0.6em;
  }

  @media (min-width: 768px) {
    .tooltip {
      padding: 16px 20px;
    }

    .grid {
      grid-template-columns: minmax(140px, 1fr) 1fr 1fr;
      grid-column-gap: 1em;
    }
  }

  .header-grid {
    align-items: end;
  }

  .beds-grid {
    background-color: #f1f1f1;
    align-items: center;
  }

  :global(.beds-grid span) {
    opacity: 0.6;
    font-style: italic;
  }

  .label {
    font-weight: 500;
  }

  .col-label {
    text-transform: uppercase;
    color: #777;
    font-size: 0.9em;
  }

  .tooltip-body li a {
    color: var(--primary);
    font-weight: 600;
    cursor: pointer;
    transition: opacity 200ms;
  }

  .tooltip-body li a:hover {
    opacity: 0.4;
  }

  .bolder {
    font-weight: bold;
  }
</style>
