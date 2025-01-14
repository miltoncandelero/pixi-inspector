<script lang="ts">
  import { createEventDispatcher } from "svelte";
  import { getBridgeContext, poll } from "./bridge-fns";
  import ReloadButton from "./ReloadButton.svelte";
  import Tree from "./Tree.svelte";
  import type { OutlinerNode } from "./types";

  const dispatch = createEventDispatcher();
  const bridge = getBridgeContext();
  const tree = poll<OutlinerNode>(
    bridge,
    "__PIXI_DEVTOOLS__.outline.tree()",
    2000
  );
  $: stage = $tree.data;
  $: error = $tree.error;

  async function expand(path: string[]) {
    await bridge(`__PIXI_DEVTOOLS__.outline.expand(${JSON.stringify(path)})`);
    tree.sync();
  }
  async function collapse(path: string[]) {
    await bridge(`__PIXI_DEVTOOLS__.outline.collapse(${JSON.stringify(path)})`);
    tree.sync();
  }
  async function activate(path: string[]) {
    await bridge(`__PIXI_DEVTOOLS__.outline.activate(${JSON.stringify(path)})`);
    tree.sync();
    dispatch("activate");
  }
  async function show(path: string[]) {
    await bridge(`__PIXI_DEVTOOLS__.outline.show(${JSON.stringify(path)})`);
    tree.sync();
  }
  async function hide(path: string[]) {
    await bridge(`__PIXI_DEVTOOLS__.outline.hide(${JSON.stringify(path)})`);
    tree.sync();
  }
</script>

{#if error}
  {error.message}
  <ReloadButton />
{/if}
{#if stage}
  <Tree
    id={stage.id}
    name={stage.name}
    leaf={stage.leaf}
    active={stage.active}
    visible={stage.visible}
    children={stage.children}
    on:expand={({ detail }) => expand(detail)}
    on:collapse={({ detail }) => collapse(detail)}
    on:activate={({ detail }) => activate(detail)}
    on:show={({ detail }) => show(detail)}
    on:hide={({ detail }) => hide(detail)}
  />
{/if}
