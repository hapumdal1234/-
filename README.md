# -
사생대회갤러리
<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>내 그림 갤러리</title>
<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/@tabler/icons-webfont@latest/dist/tabler-icons.min.css" />
<style>
  :root {
    --bg: #ffffff;
    --bg-secondary: #f5f4f1;
    --text: #1a1a1a;
    --text-secondary: #6b6a66;
    --border: rgba(0,0,0,0.12);
    --radius-md: 8px;
    --radius-lg: 12px;
    --accent: #185fa5;
    --danger: #a32d2d;
  }
  * { box-sizing: border-box; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    margin: 0;
    padding: 2rem 1rem;
    background: var(--bg-secondary);
    color: var(--text);
    line-height: 1.6;
  }
  .container {
    max-width: 800px;
    margin: 0 auto;
    background: var(--bg);
    border-radius: var(--radius-lg);
    padding: 1.5rem;
    border: 0.5px solid var(--border);
  }
  h1 {
    font-size: 20px;
    font-weight: 500;
    margin: 0;
  }
  .header {
    display: flex;
    justify-content: space-between;
    align-items: center;
    margin-bottom: 1.5rem;
  }
  button {
    border: 0.5px solid var(--border);
    background: transparent;
    border-radius: var(--radius-md);
    padding: 8px 14px;
    font-size: 14px;
    cursor: pointer;
    color: var(--text);
  }
  button:hover { background: var(--bg-secondary); }
  button:active { transform: scale(0.98); }
  #fileInput { display: none; }
  #empty {
    display: none;
    text-align: center;
    padding: 3rem 1rem;
    color: var(--text-secondary);
    border: 0.5px solid var(--border);
    border-radius: var(--radius-lg);
  }
  #empty i { font-size: 32px; }
  #grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(140px, 1fr));
    gap: 12px;
  }
  .cell {
    aspect-ratio: 1;
    overflow: hidden;
    border-radius: var(--radius-md);
    border: 0.5px solid var(--border);
    cursor: pointer;
  }
  .cell img {
    width: 100%;
    height: 100%;
    object-fit: cover;
    display: block;
  }
  #lightbox {
    display: none;
    position: fixed;
    inset: 0;
    background: rgba(0,0,0,0.6);
    align-items: center;
    justify-content: center;
    padding: 2rem;
    z-index: 100;
  }
  #lightbox-inner {
    background: var(--bg);
    border-radius: var(--radius-lg);
    padding: 1rem;
    max-width: 90%;
    max-height: 90%;
    overflow: auto;
  }
  #lightbox-inner img {
    max-width: 100%;
    max-height: 70vh;
    border-radius: var(--radius-md);
    display: block;
  }
  #lightboxName {
    margin: 8px 0;
    font-size: 13px;
    color: var(--text-secondary);
  }
  .lightbox-header {
    display: flex;
    justify-content: flex-end;
    margin-bottom: 8px;
  }
  #deleteBtn { color: var(--danger); }
</style>
</head>
<body>
<div class="container">
  <div class="header">
    <h1>내 그림 갤러리</h1>
    <div>
      <input type="file" id="fileInput" accept="image/*" multiple />
      <button id="uploadBtn"><i class="ti ti-upload"></i> 이미지 추가</button>
    </div>
  </div>

  <div id="empty">
    <i class="ti ti-photo"></i>
    <p>아직 업로드한 이미지가 없습니다</p>
  </div>

  <div id="grid"></div>
</div>

<div id="lightbox">
  <div id="lightbox-inner">
    <div class="lightbox-header">
      <button id="closeLightbox" aria-label="닫기"><i class="ti ti-x"></i></button>
    </div>
    <img id="lightboxImg" alt="" />
    <p id="lightboxName"></p>
    <button id="deleteBtn"><i class="ti ti-trash"></i> 삭제</button>
  </div>
</div>

<script>
const STORAGE_KEY = 'gallery-images';
const grid = document.getElementById('grid');
const empty = document.getElementById('empty');
const fileInput = document.getElementById('fileInput');
const uploadBtn = document.getElementById('uploadBtn');
const lightbox = document.getElementById('lightbox');
const lightboxImg = document.getElementById('lightboxImg');
const lightboxName = document.getElementById('lightboxName');
const closeLightbox = document.getElementById('closeLightbox');
const deleteBtn = document.getElementById('deleteBtn');

let images = [];
let activeIndex = null;

function loadImages() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    images = raw ? JSON.parse(raw) : [];
  } catch (e) {
    images = [];
  }
  render();
}

function saveImages() {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(images));
  } catch (e) {
    console.error('저장 오류:', e);
    alert('이미지를 저장하는 중 오류가 발생했습니다. 용량이 너무 클 수 있습니다.');
  }
}

function render() {
  grid.innerHTML = '';
  if (images.length === 0) {
    empty.style.display = 'block';
    grid.style.display = 'none';
    return;
  }
  empty.style.display = 'none';
  grid.style.display = 'grid';

  images.forEach((img, i) => {
    const cell = document.createElement('div');
    cell.className = 'cell';
    const image = document.createElement('img');
    image.src = img.data;
    image.alt = img.name;
    image.addEventListener('click', () => openLightbox(i));
    cell.appendChild(image);
    grid.appendChild(cell);
  });
}

function openLightbox(i) {
  activeIndex = i;
  lightboxImg.src = images[i].data;
  lightboxImg.alt = images[i].name;
  lightboxName.textContent = images[i].name;
  lightbox.style.display = 'flex';
}

closeLightbox.addEventListener('click', () => {
  lightbox.style.display = 'none';
  activeIndex = null;
});

lightbox.addEventListener('click', (e) => {
  if (e.target === lightbox) {
    lightbox.style.display = 'none';
    activeIndex = null;
  }
});

deleteBtn.addEventListener('click', () => {
  if (activeIndex === null) return;
  images.splice(activeIndex, 1);
  saveImages();
  lightbox.style.display = 'none';
  activeIndex = null;
  render();
});

uploadBtn.addEventListener('click', () => fileInput.click());

fileInput.addEventListener('change', (e) => {
  const files = Array.from(e.target.files || []);
  let pending = files.length;
  if (pending === 0) return;

  files.forEach((file) => {
    const reader = new FileReader();
    reader.onload = () => {
      images.push({ name: file.name, data: reader.result });
      pending--;
      if (pending === 0) {
        saveImages();
        fileInput.value = '';
        render();
      }
    };
    reader.readAsDataURL(file);
  });
});

loadImages();
</script>
</body>
</html>
