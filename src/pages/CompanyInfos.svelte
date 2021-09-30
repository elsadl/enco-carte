<script>
  import Graph from "./Graph.svelte";

  import { capitalizeFirstLetter } from "../Utils.svelte";

  import { json } from "d3-fetch";

  import { dictionary, locale, _ } from "svelte-i18n";
  import { get } from "svelte/store";

  export let data;

  let dictPays;

  let snapshot_rows = [
    "revenue",
    "net_profit",
    "type",
    "facilities",
    "employees",
    "total_beds",
  ];

  json("../data/dict_pays.json").then((data) => {
    dictPays = data;
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

  function stripCountryName(string) {
    return string
      .toLowerCase()
      .replace(/ *\([^)]*\) */g, "")
      .replace(" ", "_");
  }
</script>

<div id="global-container">
  <div id="entreprise-title">
    <h1>{data.name}</h1>
    <a href={data.website}>{$_("visit_website")}</a>
  </div>
  {#if dictPays}
    <div id="main-container">
      <div id="infos-container">
        <div class="tableau" id="snapshot">
          <h3>{$_("snapshot")}</h3>
          <div>
            {#key $locale}
              {#each snapshot_rows as el}
                <div class="row" id={el}>
                  <p class="label">{$_(el)}</p>
                  <p class="value">
                    {@html data[el] ? getTranslation(data[el]) : "—"}
                  </p>
                </div>
              {/each}
              <div class="row" id="active_in">
                <p class="label">{$_("active_in")}</p>
                <p class="value">
                  {#each data["active_in"] as country}
                    {dictPays[stripCountryName(country)]
                      ? dictPays[stripCountryName(country)]["name_" + $locale] +
                        (country.match(/ *\([^)]*\) */g)
                          ? country.match(/ *\([^)]*\) */g)
                          : "")
                      : country}
                    <br />
                  {/each}
                </p>
              </div>
            {/key}
          </div>
        </div>
        <div class="tableau" id="shareholders">
          <h3>{$_("shareholders")}</h3>
          <div>
            {#key $locale}
              {#each data.shareholders as el}
                {#if el.shares}
                  <div class="row">
                    <p class="label">{getTranslation(el.name)}</p>
                    <p class="value">{el.shares.toFixed(2)} %</p>
                  </div>
                {:else}
                  <div class="row shareholder-info">
                    <p class="label">{data.translations ? data.translations["shareholders_" + $locale] : el.name}</p>
                  </div>
                {/if}
              {/each}
            {/key}
          </div>
        </div>
        {#if data.ressources[0] != ""}
          <div id="ressources">
            <h3>{$_("ressources")}</h3>
            <ul>
              {#each data.ressources as el}
                <li>
                  <a href={el}>{el}</a>
                </li>
              {/each}
            </ul>
          </div>
        {/if}
      </div>
      <div id="graph-container">
        <h3>{$_("evolution")}</h3>
        <Graph
          data={data.evolution}
          graphID="1"
          cat1="revenue"
          cat2="equity"
          unit=" €"
        />
        <Graph
          data={data.evolution}
          graphID="2"
          cat1="operating_margin"
          cat2="profit_rate"
          unit=" %"
        />
      </div>
    </div>
  {/if}
  <div id="sources" />
</div>

<style>
  p {
    margin: 0;
    padding: 0.2em;
  }

  ul {
    list-style: none;
  }

  a {
    font-weight: 500;
    color: var(--primary);
    text-decoration: none;
  }

  #global-container {
    margin-top: 1em;
  }

  #entreprise-title h1 {
    color: var(--primary);
    font-size: 2.6em;
    font-weight: 800;
  }

  #entreprise-title a::before {
    content: "→";
    padding-right: 0.4em;
  }

  #graph-container {
    margin-top: 1.4em;
  }

  #main-container > div > div {
    margin-top: 1.4em;
  }

  @media (min-width: 992px) {
    #main-container {
      display: grid;
      grid-template-columns: 4fr 3fr;
      grid-column-gap: 4em;
    }
  }

  #main-container > div h3 {
    font-size: 1.8em;
    margin-bottom: 1em;
    position: relative;
  }

  #main-container > div h3::after {
    content: "";
    position: absolute;
    height: 5px;
    width: 5rem;
    bottom: calc(-1em / 4 - 5px / 2);
    left: 0;
    background-color: var(--primary);
  }

  #infos-container #snapshot .label {
    text-transform: uppercase;
    font-weight: 500;
    color: var(--primary);
  }

  #infos-container #shareholders .value {
    font-weight: 500;
  }

  #graph-container {
    max-width: 500px;
  }

  .row {
    display: grid;
    grid-template-columns: 1fr 1fr;
  }

  #snapshot > div,
  #shareholders > div {
    background-color: var(--light);
    padding: 1em;
  }

  #shareholders > div {
    display: flex;
    flex-direction: column-reverse;
  }

  .tableau .row {
    grid-template-columns: 1fr 1fr;
    grid-column-gap: 2em;
    padding-bottom: 0.3em;
    margin-bottom: 0.3em;
    border-bottom: 1px dashed #00000050;
  }

  .tableau .row:last-child {
    padding-bottom: 0;
    margin-bottom: 0;
    border-bottom: 0;
  }

  .tableau .row .value {
    align-self: center;
  }

  :global(#snapshot .row .value.negative) {
    color: var(--primary);
  }

  #shareholders .row {
    grid-template-columns: 3fr 1fr;
  }

  #shareholders .row:last-child {
    padding-bottom: 0.3em;
    margin-bottom: 0.3em;
    border-bottom: 1px dashed #00000050;
  }

  #shareholders .row:first-child,
  #shareholders .row:only-child {
    padding-bottom: 0;
    margin-bottom: 0;
    border-bottom: 0;
  }

  #shareholders .row.shareholder-info:not(:only-child) {
    font-style: italic;
  }

  #shareholders .row.shareholder-info {
    grid-template-columns: 1fr;
  }

  #shareholders .row .value {
    justify-items: end;
    text-align: right;
  }

  #ressources li {
    margin-bottom: 0.4em;
  }

  #ressources li:before {
    content: "→";
    padding-right: 0.2em;
    color: black;
  }
</style>
