<script context="module">
  import { writable } from "svelte/store";

  const routes = {};

  export const activeRoute = writable({});

  export function register(route) {
    routes[route.path] = route;
  }
</script>

<script>
  import page from "page";
  import { onMount, onDestroy } from "svelte";

  const setupPage = () => {
    console.log("route?");
    for (let [path, route] of Object.entries(routes)) {
      page(
        path,
        (context) => ($activeRoute = { ...route, params: context.params })
      );
    }

    page.start();
  };

  onMount(setupPage);

  onDestroy(page.stop);
</script>

<slot />
