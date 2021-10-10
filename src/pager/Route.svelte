<script>
  import { baseurl } from "../stores";

  import { register, activeRoute } from "./Router.svelte";

  export let path = baseurl;
  export let component = null;

  let params = {};

  register({ path, component });

  $: if ($activeRoute.path === path) {
    params = $activeRoute.params;
  }
</script>

{#if $activeRoute.path === path}
  {#if $activeRoute.component}
    <svelte:component
      this={$activeRoute.component}
      {...$$restProps}
      {...params}
    />
  {:else}
    <slot {params} />
  {/if}
{/if}
