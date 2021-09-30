<script>
  import { geoPath, geoIdentity } from "d3-geo";
  import { json } from "d3-fetch";
  import { feature } from "topojson-client";
  import { select, selectAll } from "d3-selection";
  import { countrySelected } from "../stores.js";
  import TooltipInfos from "./map/TooltipInfos.svelte";
  import LocaleSwitcher from "./LocaleSwitcher.svelte";

  import { locale, _ } from "svelte-i18n";

  let width;
  $: height = width / 1.2;

  let countriesEU;
  let otherCountries;
  let projection;
  let path;

  let hiddenCountries = ["GL", "MA", "DZ", "TN", "EG"];

  json("../data/nuts_with_data.json").then((json) => {
    countriesEU = feature(json, json.objects.nutsrg);
    console.log(countriesEU);

    otherCountries = feature(json, json.objects.cntrg);
  });

  $: projection = geoIdentity().reflectY(true).fitWidth(width, countriesEU);

  $: path = geoPath().projection(projection);

  function selectCountry(event) {
    if (event.target.id == $countrySelected) {
      unselectCountry();
      return;
    }
    countrySelected.set(event.target.id);
  }

  function mapOnClick(event) {
    if (
      event.target.classList.contains("interactive") ||
      event.target.classList.contains("to-company") ||
      event.target.closest("#map-tooltip")
    ) {
      return;
    }
    unselectCountry();
  }

  function unselectCountry() {
    countrySelected.set("");
  }
</script>

<div id="map-container" style="min-height: {height}px" on:click={mapOnClick}>
  <LocaleSwitcher
    on:locale-changed={(e) => locale.set(e.detail)}
  />
  {#if countriesEU}
    <div id="map-wrapper" bind:offsetWidth={width}>
      <svg {width} {height}>
        {#each otherCountries.features as feature}
          {#if !hiddenCountries.includes(feature.properties.id)}
            <path
              id={feature.properties.id}
              d={path(feature)}
              class="country not-interactive"
            />
          {/if}
        {/each}
        {#each countriesEU.features as feature}
          {#if !feature.properties.data}
            <path
              id={feature.properties.id}
              d={path(feature)}
              class="country not-interactive"
            />
          {:else}
            <path
              id={feature.properties.id}
              d={path(feature)}
              class="country interactive"
              class:selected={$countrySelected === feature.properties.id}
              on:click={selectCountry}
            />
          {/if}
        {/each}
      </svg>
    </div>

    <div id="sources">
      <p><span>{$_("sources")}</span> {$_("map_sources")}</p>
    </div>
    <div id="map-tooltip" style="min-height: {height}px">
      {#if $countrySelected}
        {#each countriesEU.features as feature}
          {#if $countrySelected == feature.properties.id}
            <TooltipInfos country={feature.properties} />
          {/if}
        {/each}
      {:else}
        <div id="map-text">
          <h1 id="map-title">{$_("map_title")}</h1>
          <p id="map-info">{$_("map_info")}</p>
        </div>
      {/if}
    </div>
  {/if}
</div>

<style>
  #map-container {
    position: relative;
  }

  #map-wrapper {
    overflow: hidden;
    z-index: 1;
    font-size: 0;
  }

  #map-wrapper svg {
    transform: translateX(5%);
  }

  #map-tooltip {
    position: relative;
    z-index: 2;
    padding-top: 2em;
  }

  #map-title {
    font-size: 2.6em;
    font-weight: 900;
    color: var(--primary);
  }

  #map-info {
    font-size: 1.2em;
    margin-top: 0.6em;
    color: #000;
  }

  #sources {
    font-size: 0.85em;
    padding-top: 1.2em;
    text-align: right;
  }

  #sources span {
    font-weight: 600;
  }

  @media (max-width: 600px) {
    #map-text, #map-tooltip {
      font-size: 0.85em;
    }
  }

  @media (min-width: 992px) {
    #map-wrapper {
      position: absolute;
      width: 70%;
      top: 0;
      right: 0;
    }

    #map-tooltip {
      max-width: 456px;
    }

    #map-wrapper svg {
      transform: translateX(15%);
    }

    #sources {
      position: absolute;
      right: 0;
      bottom: -2em;
      max-width: 75%;
    }
  }

  .country:focus {
    outline: 0;
  }

  .country {
    stroke: grey;
    stroke-width: 0.6;
  }

  .country.interactive {
    fill: #f7f7f7;
    cursor: pointer;
    transition: fill 0.3s;
  }

  .country.interactive:hover {
    fill: #fff;
  }

  .country.selected,
  .country.selected:hover {
    fill: var(--primary);
  }

  .country.not-interactive {
    fill: #cecece;
  }
</style>
