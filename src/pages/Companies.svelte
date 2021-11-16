<script>
  import { _ } from "svelte-i18n";
  import { baseurl } from "../stores";

  export let entreprises;
  entreprises.sort((a, b) => a.nom.localeCompare(b.nom));
</script>

<div id="footer">
  <h2>{$_("companies_list")}</h2>
  <ul id="liste-entreprises">
    {#each entreprises as entreprise}
      <li>
        <a class="to-company" href={baseurl + "company/" + entreprise.id}
          >{entreprise.nom}</a
        >
      </li>
    {/each}
  </ul>
</div>

<style>
  #footer {
    background-color: var(--light);
    padding: 1em;
    margin-top: 3em;
    border-top: 5px solid var(--primary);
  }

  #footer h2 {
    margin-bottom: 0.8em;
    color: var(--primary);
  }

  #liste-entreprises {
    column-count: 1;
  }

  @media (min-width: 490px) {
    #liste-entreprises {
      grid-template-columns: 1fr 1fr;
    }
  }

  @media (min-width: 576px) {
    #footer {
      padding: 2em;
      margin-top: 4em;
    }
  }

  @media (min-width: 992px) {
    #footer {
      padding: 4em;
      margin-top: 6em;
    }
    #liste-entreprises {
      column-count: 3;
    }
  }

  @media (min-width: 1050px) {
    #liste-entreprises {
      column-count: 4;
    }
  }

  #liste-entreprises a {
    color: var(--primary);
  }
</style>
