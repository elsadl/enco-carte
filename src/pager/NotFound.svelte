<script>
  import { baseurl } from "../stores";

  import { register, activeRoute } from "./Router.svelte";

  export let path = baseurl + "*";
  export let component = null;

  register({ path, component });
</script>

{#if $activeRoute.path === path}
  <svelte:component this={component} />
  <slot />
{/if}
