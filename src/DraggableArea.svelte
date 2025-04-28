<!--
子要素の重なり順の面倒を見るコンポーネント
-->
<script>
  import { setContext } from "svelte";

  let area;
  let items = [];
  setContext("area", {
    add: (item) => {
      items.push(item);
    },
    toFront: (item) => {
      items = items.filter((c) => c !== item);
      Array.prototype.push.apply(items, [item]);
      items.forEach((item, i) => {
        item.style.zIndex = i + 1;
      });
    },
    remove: (item) => {
      items = items.filter((c) => c !== item);
    },
    getWidth: () => {
      return area.clientWidth;
    },
    getHeight: () => {
      return area.clientHeight;
    },
  });
</script>

<div bind:this={area}>
  <slot />
</div>

<style>
  div {
    position: relative;
  }
</style>
