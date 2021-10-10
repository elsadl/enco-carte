<script>
  import Router from "./pager/Router.svelte";
  import Route from "./pager/Route.svelte";
  import NotFound from "./pager/NotFound.svelte";

  import { addMessages, init, locale, _ } from "svelte-i18n";

  import Company from "./pages/Company.svelte";
  import Map from "./pages/Map.svelte";

  import en from "../public/lang/en.json";
  import fr from "../public/lang/fr.json";
import { baseurl } from "./stores";

  addMessages("en", en);
  addMessages("fr", fr);

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
