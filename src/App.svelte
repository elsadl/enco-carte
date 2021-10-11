<script>
  import Router from "./pager/Router.svelte";
  import Route from "./pager/Route.svelte";
  import NotFound from "./pager/NotFound.svelte";

  import { addMessages, init, locale, _ } from "svelte-i18n";

  import Company from "./pages/Company.svelte";
  import Map from "./pages/Map.svelte";

  import en from "../public/lang/en.json";
  import es from "../public/lang/es.json";
  import fr from "../public/lang/fr.json";
  import de from "../public/lang/de.json";
  import it from "../public/lang/it.json";

  import { baseurl } from "./stores";

  addMessages("en", en);
  addMessages("es", es);
  addMessages("fr", fr);
  addMessages("de", de);
  addMessages("it", it);

  init({
    initialLocale: "fr",
  });
</script>

<div class="container">
  <Router>
    <Route path={baseurl} component={Map} />
    <Route path={baseurl + "company/:id"} let:params>
      {#key params.id}
        <Company id={params.id} />
      {/key}
    </Route>
    <NotFound>
      <h2>{$_("page_not_found")}.</h2>
    </NotFound>
  </Router>
</div>

<style>
</style>
