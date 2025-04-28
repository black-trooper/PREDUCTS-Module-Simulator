<!--
子要素をドラッグ可能にするコンポーネント
-->
<script>
  import { onMount, getContext, createEventDispatcher } from "svelte";
  export let scale;
  export let x = 0;
  export let y = 0;
  export let width;
  export let height;
  export let ignoreElements = ""; // 無視するセレクタ
  export let selectedDesk;
  export let module;

  const dispatch = createEventDispatcher();
  let areaContext = getContext("area");
  let item;
  const position = { x: parseInt(x), y: parseInt(y) }; // 要素の位置
  const diff = { x: 0, y: 0 }; // ドラッグ開始時の要素の位置とマウスの位置の差
  const areaMargin = 20;
  let isCatched = false;

  function handleMove(event) {
    if (isCatched) {
      const point = event.touches ? event.touches[0] : event;
      position.x = parseFloat(point.clientX / scale) - diff.x;
      position.y = parseFloat(point.clientY / scale) - diff.y;
    }
  }

  // position が変更されたら領域内に収まるように調整する
  $: {
    position; // これを記載しないと position が変更されても stickInArea() は実行されない。
    stickInArea();
    snapToRail();
    snapToHole();
  }

  function stickInArea() {
    if (!item) {
      return;
    }
    // エリアをはみ出さないように調整
    if (position.x < -areaMargin) {
      position.x = -areaMargin;
    } else if (position.x > selectedDesk.width - width + areaMargin) {
      position.x = selectedDesk.width - width + areaMargin;
    }
    if (position.y < -areaMargin) {
      position.y = -areaMargin;
    } else if (position.y > selectedDesk.height - height + areaMargin) {
      position.y = selectedDesk.height - height + areaMargin;
    }
    position.x = parseInt(position.x);
    position.y = parseInt(position.y);
  }

  function handleCatch(event) {
    if (ignoreElements && event.target.closest(ignoreElements)) {
      return;
    }
    isCatched = true;
    const point = event.touches ? event.touches[0] : event;
    diff.x = parseFloat(point.clientX / scale) - parseFloat(position.x);
    diff.y = parseFloat(point.clientY / scale) - parseFloat(position.y);
    areaContext.toFront(item);
  }

  function handleRelease(event) {
    isCatched = false;
    dispatch("drag", position); // ドラッグ中に親に位置を通知
  }
  function snapToRail() {
    if (!selectedDesk || !selectedDesk.rails || !module) {
      return;
    }
    const snapped = module.holes.some((hole) => {
      const holeTop = position.y + hole.y;
      const holeBottom = holeTop + hole.height;
      return selectedDesk.rails.some((rail) => {
        const railTop = rail.y;
        const railBottom = railTop + 5; // レールの高さ（例: 5px）
        return holeTop <= railTop && railBottom <= holeBottom;
      });
    });
    if (snapped) {
      return;
    }

    // モジュールのネジ穴を基準にスナップ
    module.holes.forEach((hole) => {
      const holeGlobal = {
        top: position.y + hole.y,
        bottom: position.y + hole.y + hole.height,
      };

      let closestRail = null;
      let closestDistance = Infinity;

      // 各レールに対して距離を計算
      selectedDesk.rails.forEach((rail) => {
        // レールの範囲（横幅全体）
        const railTop = rail.y;
        const railBottom = railTop + 5; // レールの高さ（例: 5px）

        let distance = Math.abs(holeGlobal.top - railBottom);
        if (distance < closestDistance) {
          closestDistance = distance;
          closestRail = { top: railTop };
        }
        distance = Math.abs(holeGlobal.bottom - railTop);
        if (distance < closestDistance) {
          closestDistance = distance;
          closestRail = { bottom: railBottom };
        }
      });

      // スナップ範囲（例: 20px）内の場合にスナップ
      const snapRange = 20;
      if (closestRail && closestDistance <= snapRange) {
        // モジュールの位置を調整
        if (closestRail.top) {
          position.y += closestRail.top - holeGlobal.top;
        } else {
          position.y += closestRail.bottom - holeGlobal.bottom;
        }
      }
    });
  }
  function snapToHole() {
    if (!selectedDesk || !selectedDesk.holes || !module) {
      return;
    }
    const snapped = module.holes.some((hole) => {
      const holeGlobal = {
        left: position.x + hole.x,
        right: position.x + hole.x + hole.width,
      };

      return selectedDesk.holes.some((hole) => {
        // レールの範囲（横幅全体）
        const serviceHoleLeft = hole.x;
        const serviceHoleRight = serviceHoleLeft + hole.diameter;
        return (
          holeGlobal.left <= serviceHoleLeft &&
          serviceHoleRight <= holeGlobal.right
        );
      });
    });
    if (snapped) {
      return;
    }

    // モジュールのネジ穴を基準にスナップ
    module.holes.forEach((hole) => {
      const holeGlobal = {
        left: position.x + hole.x,
        right: position.x + hole.x + hole.diameter,
      };

      let closestServiceHole = null;
      let closestDistance = Infinity;

      // 各サービスネジ穴に対して距離を計算
      selectedDesk.holes.forEach((hole) => {
        const serviceHoleLeft = hole.x;
        const serviceHoleRight = serviceHoleLeft + hole.diameter;

        let distance = Math.abs(holeGlobal.left - serviceHoleRight);
        if (distance < closestDistance) {
          closestDistance = distance;
          closestServiceHole = { left: serviceHoleLeft };
        }
        distance = Math.abs(holeGlobal.right - serviceHoleLeft);
        if (distance < closestDistance) {
          closestDistance = distance;
          closestServiceHole = { right: serviceHoleRight };
        }
      });

      // スナップ範囲（例: 20px）内の場合にスナップ
      const snapRange = 20;
      if (closestServiceHole && closestDistance <= snapRange) {
        // モジュールの位置を調整
        if (closestServiceHole.left) {
          position.x += closestServiceHole.left - holeGlobal.left;
        } else {
          position.x += closestServiceHole.right - holeGlobal.right;
        }
      }
    });
  }

  onMount(() => {
    areaContext.add(item);
    return () => {
      areaContext.remove(item);
    };
  });
</script>

<svelte:window
  on:mousemove={handleMove}
  on:mouseup={handleRelease}
  on:touchmove|passive={handleMove}
  on:touchend={handleRelease}
  on:resize={stickInArea}
/>

<div
  class="item"
  class:isCatched
  bind:this={item}
  style:top={position.y + "px"}
  style:left={position.x + "px"}
  on:mousedown|stopPropagation={handleCatch}
  on:touchstart|passive={handleCatch}
>
  <slot />
</div>

<style>
  .item {
    position: absolute;
    touch-action: none; /* スクロールやズームを無効化 */
    transition:
      box-shadow 0.3s,
      transform 0.3s;
    cursor: grab;
    user-select: none;
    -webkit-user-select: none; /* Safari対応 */
  }
  .item > :global(*) {
    margin: 0;
  }
  .item.isCatched > :global(*) {
    box-shadow: 0 0.25rem 1rem 0 rgba(0, 0, 0, 0.5);
  }
  .item.isCatched {
    cursor: grabbing;
  }
</style>
