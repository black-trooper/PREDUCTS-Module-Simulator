<!-- Module.svelte -->
<script>
  export let scale;
  export let module;
  export let index;
  export let updatePosition;
  export let updateCollisionStates;
  export let removeModule;
  export let selectedDesk;

  import Draggable from "./Draggable.svelte";

  let rotation = 0;
  let currentSizeIndex = 0;

  const onDrag = (event) => {
    updatePosition(index, event.detail);
  };

  const rotateModule = (radial, changeSize) => {
    if (!changeSize) {
      rotation = (rotation + radial) % 360;
    }
    for (let i = 0; i < radial / 90; i++) {
      [module.width, module.height] = [module.height, module.width]; // 回転後に幅と高さを入れ替える
      // 各穴の位置を回転に合わせて変換
      module.holes = module.holes.map((hole) => {
        let newHole = { ...hole };
        newHole = {
          x: module.width - hole.y - hole.height,
          y: hole.x,
          width: hole.height,
          height: hole.width,
        };
        return newHole;
      });
    }
    // 衝突判定を再評価
    updateCollisionStates();
  };

  const toggleModuleSize = () => {
    currentSizeIndex = (currentSizeIndex + 1) % module.install.length; // 次のサイズインデックス
    // サイズを更新
    module.width = module.install[currentSizeIndex].width;
    module.height = module.install[currentSizeIndex].height;
    module.holes = module.install[currentSizeIndex].holes;

    rotateModule(rotation, true);

    // 衝突判定を再評価
    updateCollisionStates();
  };
</script>

<Draggable
  x={module.x}
  y={module.y}
  {scale}
  width={module.width}
  height={module.height}
  on:drag={onDrag}
  ignoreElements=".rotate-btn, .delete-btn"
  {selectedDesk}
  {module}
>
  <div
    class="module {module.isValid ? 'valid' : 'invalid'}"
    style="
    width: {module.width}px;
    height: {module.height}px;
    "
  >
    <div class="module-container">
      <div class="module-content">
        <div class="module-name">
          {module.name
            .replace("Mount for DASHBOARD", "")
            .replace("for DASHBOARD", "")
            .replace("Mount for", "")}
        </div>

        <div class="module-actions">
          {#if module.install.length > 1}
            <button
              class="rotate-btn"
              on:click={toggleModuleSize}
              title="サイズ変更"
            >
              <i class="fas fa-arrows-alt-h"></i>
            </button>
          {/if}

          {#if module.rotate}
            <button
              class="rotate-btn"
              on:click={() => rotateModule(module.rotate)}
              title="回転"
            >
              <i class="fas fa-sync-alt"></i></button
            >
          {/if}
          <button
            class="delete-btn"
            on:click={() => removeModule(index)}
            title="削除"><i class="fas fa-trash"></i></button
          >
        </div>
      </div>
    </div>

    {#each module.holes as hole}
      <div
        class="hole"
        style="
        top: {hole.y - Math.min(hole.width, hole.height) / 2}px;
        left: {hole.x - Math.min(hole.width, hole.height) / 2}px;
        width: {hole.width}px;
        height: {hole.height}px;
      "
      ></div>
    {/each}
  </div>
</Draggable>

<style>
  .module-container {
    display: flex;
    justify-content: center; /* 横中央 */
    align-items: center; /* 縦中央 */
    height: 100%; /* 画面全体の高さ */
  }

  .module-content {
    display: flex;
    flex-direction: column; /* 名前とボタンを縦に配置 */
    justify-content: center; /* 中央揃え */
    align-items: center; /* 中央揃え */
    text-align: center; /* テキスト中央揃え */
  }

  .module-name {
    font-weight: bold;
    color: #333; /* 文字色 */
    margin-bottom: 10px; /* 名前とボタンの間にスペース */
    user-select: none;
    -webkit-user-select: none; /* Safari対応 */
  }

  .module-actions {
    display: flex;
    flex-wrap: wrap; /* 狭いときに折り返す */
    justify-content: center; /* ボタンを中央揃え */
    gap: 5px; /* ボタン間の間隔 */
    max-width: 100%; /* 親要素の幅を超えない */
    overflow: hidden;
  }

  .rotate-btn,
  .delete-btn {
    padding: 6px 10px;
    font-size: 10px;
    cursor: pointer;
    border: none;
    background-color: #656565; /* 回転ボタンの背景色 */
    color: white;
    border-radius: 4px;
  }

  .delete-btn {
    background-color: #d7502b; /* 削除ボタンの背景色 */
  }

  .rotate-btn:hover,
  .delete-btn:hover {
    opacity: 0.8; /* ボタンにホバー効果 */
  }

  .module {
    position: absolute;
    border: 1px solid black;
    text-align: center;
    cursor: grab;
    opacity: 70%;
  }

  .module.valid {
    background-color: #fff; /* 一致している場合 */
    border: 1px solid #222;
  }

  .module.invalid {
    background-color: #f5d4d4; /* 一致していない場合 */
    border: 1px solid #c27e7e;
  }

  .hole {
    position: absolute;
    background-color: #818181;
    border-radius: 5px;
    width: 10px;
    height: 40px;
  }
</style>
