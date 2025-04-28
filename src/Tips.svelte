<script>
  let slides = [
    {
      title: "モジュールの配置",
      text: "画面下部のボタンでモジュールを配置し、ドラッグ＆ドロップでレールやサポートネジに自由に配置できます。固定できない位置に配置するとモジュールが赤色になります",
    },
    {
      title: "モジュールの回転",
      text: "回転アイコンを押すと90度ずつ回転できます。",
    },
    {
      title: "設置タイプの変更",
      text: "PCマウントやPS5用マウントは設置タイプの変更ができます。",
    },
    {
      title: "デスクの向き",
      text: "デスク上部が前面です。前面を示すラインが引いてあります。",
    },
  ];

  let showModal = true;
  let hideTips = localStorage.getItem("hideTips") === "true";

  // モーダルを閉じる処理
  const close = () => {
    showModal = false;
    localStorage.setItem("hideTips", "true"); // ローカルストレージに保存
  };

  // モーダルが表示されるかどうかの判定
  if (hideTips) {
    showModal = false;
  }

  // 任意のタイミングでモーダルを表示する関数
  const showTipsAgain = () => {
    showModal = true;
  };
</script>

{#if showModal}
  <div class="modal">
    <div class="content">
      <h1 class="modal-title">How to use</h1>

      <!-- Tips一覧表示 -->
      <div class="tips-list">
        {#each slides as slide}
          <div class="tip-item">
            <h2>{slide.title}</h2>
            <p>{slide.text}</p>
          </div>
        {/each}
      </div>

      <div class="controls">
        <button on:click={close}>Close</button>
      </div>
    </div>
  </div>
{/if}

<!-- 任意のタイミングでモーダルを再表示するボタン -->
<button on:click={showTipsAgain} class="help-btn">Help</button>

<style>
  .modal {
    position: fixed;
    inset: 0;
    background: rgba(0, 0, 0, 0.6);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 999;
  }
  .content {
    background: #fff;
    padding: 20px;
    width: 90%;
    max-width: 600px;
    border-radius: 8px;
    text-align: left;
  }
  h1.modal-title {
    font-size: 18px; /* フォントサイズを調整 */
    color: #333;
    margin-bottom: 20px;
  }
  h2 {
    font-size: 16px;
    margin: 12px 0 8px;
  }
  p {
    font-size: 14px;
    color: #444;
    margin-bottom: 20px;
  }
  .tips-list {
    margin-bottom: 20px;
  }
  .tip-item {
    margin-bottom: 15px;
  }
  .controls {
    margin-top: 20px;
    text-align: center;
  }
  .help-btn {
    margin: 0;
  }
</style>
