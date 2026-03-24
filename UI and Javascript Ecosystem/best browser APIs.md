# The Browser Is Already a Supercomputer. You Just Have to Ask.
**The 10 best browser APIs we can use without installing libraries**

Developers spend hours configuring build tools, auditing dependencies, and debating bundle sizes, all to ship features that have been sitting inside the browser, fully built, completely free, waiting to be used. No npm install. No import from a CDN. No third-party trust required.

These are ten of those features. Each one is production-ready, widely supported, and powerful enough to replace a library you might be reaching for today.

Here are all the working examples just for you:

---

### 1. Fetch + Streams API
Fetch is the standard for HTTP requests. Combined with the Streams API, you can process data as it arrives, which is exactly how AI token streaming works.



```html
<script>
  const res = await fetch('/api/data');
  const reader = res.body.getReader();
  const decoder = new TextDecoder();
  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    console.log(decoder.decode(value, { stream: true }));
  }
</script>
```

---

### 2. Web Workers
JavaScript runs on one thread. Web Workers give you a second one. Heavy tasks like parsing, encryption, or data processing run in the background without freezing the UI.



```html
<script>
  // Worker code must live in its own scope — use a Blob URL to inline it
  const code = `
    self.onmessage = ({ data }) => {
      const result = data.map(n => n * 2);
      self.postMessage(result);
    };
  `;
  const blob = new Blob([code], { type: 'application/javascript' });
  const worker = new Worker(URL.createObjectURL(blob));
  worker.postMessage([1, 2, 3, 4, 5]);
  worker.onmessage = (e) => console.log('Done:', e.data); // [2, 4, 6, 8, 10]
</script>
```

---

### 3. Intersection Observer
Fires a callback when an element enters or leaves the viewport. The right way to handle lazy loading and scroll animations, with no scroll event listeners needed.



```html
<script>
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.classList.add('visible');
        observer.unobserve(entry.target);
      }
    });
  }, { threshold: 0.2 });
  document.querySelectorAll('.card').forEach(el => observer.observe(el));
</script>
```

---

### 4. IndexedDB
A full async database in the browser. Stores structured data, blobs, and files. Far beyond what localStorage can offer.



```html
<script>
  const request = indexedDB.open('AppDB', 1);
  request.onupgradeneeded = (e) => {
    e.target.result.createObjectStore('notes', { autoIncrement: true });
  };
  request.onsuccess = (e) => {
    const db = e.target.result;
    const tx = db.transaction('notes', 'readwrite');
    tx.objectStore('notes').add({ text: 'Hello', date: Date.now() });
  };
</script>
```

---

### 5. WebSockets
A persistent, two-way connection between browser and server. Either side can send a message at any time. The foundation of chat, live dashboards, and multiplayer.

```html
<script>
  const ws = new WebSocket('wss://your-server.com/socket');
  ws.onopen    = () => ws.send(JSON.stringify({ type: 'hello' }));
  ws.onmessage = (e) => console.log(JSON.parse(e.data));
  ws.onclose   = () => console.log('Disconnected');
</script>
```

---

### 6. File System Access API
Read and write real files on the user’s disk, with their permission. Enables desktop-class editors and tools that run entirely in the browser.

```html
<script>
  async function openAndSave() {
    const [handle] = await window.showOpenFilePicker();
    const file = await handle.getFile();
    const text = await file.text();
    const edited = text + '\n// edited';
    const writable = await handle.createWritable();
    await writable.write(edited);
    await writable.close();
  }
</script>
```

---

### 7. Canvas and OffscreenCanvas
Canvas gives you pixel-level drawing. OffscreenCanvas moves rendering into a Worker so graphics never compete with the main thread.

```html
<canvas id="c" width="400" height="200"></canvas>
<script>
  const ctx = document.getElementById('c').getContext('2d');
  function draw(t) {
    ctx.clearRect(0, 0, 400, 200);
    ctx.fillStyle = '#c8f542';
    ctx.beginPath();
    ctx.arc(200 + Math.sin(t / 500) * 150, 100, 20, 0, Math.PI * 2);
    ctx.fill();
    requestAnimationFrame(draw);
  }
  requestAnimationFrame(draw);
</script>
```

---

### 8. MediaRecorder + getUserMedia
Access the camera, microphone, or screen and record whatever comes through. No third-party SDK required.

```html
<audio id="playback" controls></audio>
<script>
  const audio = document.getElementById('playback');
  const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
  const recorder = new MediaRecorder(stream);
  const chunks = [];
  recorder.ondataavailable = (e) => chunks.push(e.data);
  recorder.onstop = () => {
    const blob = new Blob(chunks, { type: 'audio/webm' });
    audio.src = URL.createObjectURL(blob);
  };
  recorder.start();
  setTimeout(() => recorder.stop(), 5000);
</script>
```

---

### 9. requestAnimationFrame + Performance API
`requestAnimationFrame` syncs your code with the screen refresh rate. The Performance API measures elapsed time with sub-millisecond precision. Together they power smooth, accurate animation.

```html
<canvas id="c" width="600" height="80"></canvas>
<script>
  const canvas = document.getElementById('c');
  const ctx = canvas.getContext('2d');
  let x = 0;
  let last = 0;
  function loop(timestamp) {
    const delta = timestamp - last;
    last = timestamp;
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    x = (x + 200 * (delta / 1000)) % canvas.width;
    ctx.beginPath();
    ctx.arc(x, canvas.height / 2, 16, 0, Math.PI * 2);
    ctx.fillStyle = '#c8f542';
    ctx.fill();
    requestAnimationFrame(loop);
  }
  requestAnimationFrame(loop);
</script>
```

---

### 10. Broadcast Channel API
Lets tabs, windows, and workers from the same origin communicate instantly. Sync auth state, share cache updates, or coordinate UI across multiple open tabs, with no server involved.

```html
<script>
  const channel = new BroadcastChannel('app');
  // Send from any tab
  channel.postMessage({ type: 'logout' });
  // Receive in every other tab
  channel.onmessage = ({ data }) => {
    if (data.type === 'logout') redirectToLogin();
  };
</script>
```

---

### The Point
None of these need npm. None add to your bundle. They are built into every modern browser and have been for years. Before reaching for a library, it is worth asking whether the platform already has what you need. Often, it does.