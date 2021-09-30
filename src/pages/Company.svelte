<script>
  import CompanyInfos from "./CompanyInfos.svelte";
  import Companies from "./Companies.svelte";
  import LocaleSwitcher from "./LocaleSwitcher.svelte";

  import { json } from "d3-fetch";

  import { locale, _ } from "svelte-i18n";

  export let id;

  let dataToDisplay = {};
  let entreprises;

  json("../data/entreprises.json").then((data) => {
    entreprises = data;
    if (data.find((el) => el.id == id)) {
      prepareData(data.find((el) => el.id == id));
    }
  });

  function prepareData(entreprise) {
    dataToDisplay["name"] = entreprise.nom;
    dataToDisplay["id"] = entreprise.id;
    dataToDisplay["country"] = entreprise.country;
    dataToDisplay["shareholders"] = entreprise.shareholders;
    dataToDisplay["translations"] = entreprise.translations;
    dataToDisplay["active_in"] = entreprise.active_in;
    dataToDisplay["ressources"] = entreprise.ressources;
    dataToDisplay["website"] = entreprise.website;

    console.log(dataToDisplay["ressources"]);
    if (dataToDisplay["ressources"][0] == " ") {
      dataToDisplay["ressources"] = null;
    }

    dataToDisplay["evolution"] = [];
    for (let i = 0; i < Object.keys(entreprise.evolution).length; i++) {
      dataToDisplay["evolution"][i] = Object.values(entreprise.evolution)[i];
    }
    let lastYear = Object.keys(entreprise.evolution)[
      Object.keys(entreprise.evolution).length - 1
    ];
    for (let i = lastYear; i > Object.keys(entreprise.evolution)[0] - 1; i--) {
      if (entreprise.evolution[i]["revenue"]) {
        dataToDisplay["revenue"] = `${entreprise.evolution[i][
          "revenue"
        ].toLocaleString("de-DE")} € <span>(${i})</span>`;
        break;
      } else {
        dataToDisplay["revenue"] = "—";
      }
    }
    for (let i = lastYear; i > Object.keys(entreprise.evolution)[0] - 1; i--) {
      if (entreprise.evolution[i]["net_profit"]) {
        dataToDisplay["net_profit"] = `${entreprise.evolution[i][
          "net_profit"
        ].toLocaleString("de-DE")} € <span>(${i})</span>`;
        break;
      } else {
        dataToDisplay["net_profit"] = "—";
      }
    }

    dataToDisplay["type"] = entreprise.type;
    dataToDisplay["facilities"] =
      entreprise.facilities != " "
        ? entreprise.facilities.toLocaleString("de-DE")
        : "—";
    dataToDisplay["total_beds"] =
      entreprise.total_beds != " "
        ? entreprise.total_beds.toLocaleString("de-DE")
        : "—";
    dataToDisplay["employees"] =
      entreprise.employees != " "
        ? entreprise.employees.toLocaleString("de-DE")
        : "—";
  }
</script>

<div>
  <div class="nav">
    <LocaleSwitcher
      on:locale-changed={(e) => locale.set(e.detail)}
    />
    <a href="/" class="back">{$_("back_map")} →</a>
  </div>

  {#if dataToDisplay["evolution"]}
    <CompanyInfos data={dataToDisplay} />
  {:else if id == "undefined"}
    <p>{$_("company_not_found")}</p>
  {/if}

  {#if entreprises}
    <Companies {entreprises} />
  {/if}
</div>

<style>
  .back {
    margin-bottom: 2em;
    display: inline-block;
    padding-bottom: 0.1em;
    color: black;
    font-weight: 600;
    font-size: 0.9em;
    text-transform: uppercase;
    border-bottom: 2px solid transparent;
  }

  .back:hover {
    border-bottom: 2px solid var(--primary);
    opacity: 1;
  }

  .nav {
    display: flex;
    justify-content: space-between;
  }
</style>
