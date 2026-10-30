<!DOCTYPE html>
<html lang="ja">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>JUMPX - バスケットボールジャンプフォーム比較</title>
  <style>
    :root {
      --bg-color: #0d1b2a;
      --card-bg: #1b263b;
      --accent-color: #00b4d8;
      --text-color: #ffffff;
      --border-color: #415a77;
    }

    body {
      margin: 0;
      padding: 20px;
      background-color: var(--bg-color);
      color: var(--text-color);
      font-family: sans-serif;
    }

    h1 {
      text-align: center;
      color: var(--accent-color);
      margin-bottom: 20px;
    }

    .global-controls {
      max-width: 1000px;
      margin: 0 auto 20px auto;
      background: var(--card-bg);
      padding: 15px;
      border-radius: 8px;
      display: flex;
      justify-content: center;
      gap: 15px;
      flex-wrap: wrap;
      align-items: center;
    }

    .container {
      max-width: 1000px;
      margin: 0 auto;
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 20px;
    }

    @media (max-width: 700px) {
      .container {
        grid-template-columns: 1fr;
      }
    }

    .player-card {
      background: var(--card-bg);
      padding: 15px;
      border-radius: 8px;
      border: 1px solid var(--border-color);
    }

    .video-wrapper {
      position: relative;
      width: 100%;
      background: #000;
      margin: 10px 0;
      border-radius: 4px;
      overflow: hidden;
    }

    video {
      width: 100%;
      max-height: 400px;
      display: block;
    }

    .controls {
      display: flex;
      flex-direction: column;
      gap: 8px;
    }

    .btn-group {
      display: flex;
      gap: 5px;
      flex-wrap: wrap;
    }

    button, input[type="file"]::file-selector-button {
      background: var(--accent-color);
      color: #000;
      border: none;
      padding: 8px 12px;
      border-radius: 4px;
      font-weight: bold;
      cursor: pointer;
    }

    input[type="range"] {
      width: 100%;
    }
  </style>
</head>
<body>

  <h1>JUMPX</h1>

  <div class="global-controls">
    <button id="masterPlay">▶ 同時再生 / 停止</button>
    <button id="masterReset">↺ 基準（Sync）へ戻る</button>
    <label>
      速度:
      <select id="speedSelect">
        <option value="0.25">0.25x</option>
        <option value="0.5">0.5x</option>
        <option value="1.0" selected>1.0x</option>
      </select>
    </label>
  </div>

  <div class="container">
    <!-- 自分 -->
    <div class="player-card">
      <h3>自分の動画</h3>
      <input type="file" id="file1" accept="video/*">
      <div class="video-wrapper">
        <video id="video1" controls></video>
      </div>
      <div class="controls">
        <input type="range" id="slider1" value="0" min="0" max="100" step="0.1">
        <div class="btn-group">
          <button id="setSync1">踏み切り（Sync）設定</button>
          <button id="prev1">◀ 1コマ戻る</button>
          <button id="next1">1コマ進む ▶</button>
        </div>
        <span id="sync1Text">Sync: 未設定</span>
      </div>
    </div>

    <!-- お手本 -->
    <div class="player-card">
      <h3>お手本動画</h3>
      <input type="file" id="file2" accept="video/*">
      <div class="video-wrapper">
        <video id="video2" controls></video>
      </div>
      <div class="controls">
        <input type="range" id="slider2" value="0" min="0" max="100" step="0.1">
        <div class="btn-group">
          <button id="setSync2">踏み切り（Sync）設定</button>
          <button id="prev2">◀ 1コマ戻る</button>
          <button id="next2">1コマ進む ▶</button>
        </div>
        <span id="sync2Text">Sync: 未設定</span>
      </div>
    </div>
  </div>

  <script>
    const setup = (id) => {
      const v = document.getElementById(`video${id}`);
      const f = document.getElementById(`file${id}`);
      const s = document.getElementById(`slider${id}`);
      const btnSync = document.getElementById(`setSync${id}`);
      const txtSync = document.getElementById(`sync${id}Text`);
      let syncTime = 0;

      f.addEventListener('change', (e) => {
        const file = e.target.files[0];
        if (file) {
          v.src = URL.createObjectURL(file);
        }
      });

      v.addEventListener('timeupdate', () => {
        if (v.duration) {
          s.value = (v.currentTime / v.duration) * 100;
        }
      });

      s.addEventListener('input', () => {
        if (v.duration) {
          v.currentTime = (s.value / 100) * v.duration;
        }
      });

      btnSync.addEventListener('click', () => {
        syncTime = v.currentTime;
        txtSync.textContent = `Sync: ${syncTime.toFixed(2)}s`;
      });

      return { v, getSync: () => syncTime, setTime: (t) => v.currentTime = t };
    };

    const p1 = setup(1);
    const p2 = setup(2);

    document.getElementById('masterPlay').addEventListener('click', () => {
      if (p1.v.paused || p2.v.paused) {
        p1.v.play();
        p2.v.play();
      } else {
        p1.v.pause();
        p2.v.pause();
      }
    });

    document.getElementById('masterReset').addEventListener('click', () => {
      p1.setTime(p1.getSync());
      p2.setTime(p2.getSync());
    });

    document.getElementById('speedSelect').addEventListener('change', (e) => {
      const spd = parseFloat(e.target.value);
      p1.v.playbackRate = spd;
      p2.v.playbackRate = spd;
    });

    const step = 1 / 30;
    document.getElementById('prev1').addEventListener('click', () => p1.v.currentTime -= step);
    document.getElementById('next1').addEventListener('click', () => p1.v.currentTime += step);
    document.getElementById('prev2').addEventListener('click', () => p2.v.currentTime -= step);
    document.getElementById('next2').addEventListener('click', () => p2.v.currentTime += step);
  </script>
</body>
</html>
