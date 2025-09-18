# Birthday-slideshow

<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Happy Birthday — Slideshow Reel</title>
  <style>
    /* Basic reset */
    *{box-sizing:border-box;margin:0;padding:0}
    html,body{height:100%}
    body{font-family:system-ui,-apple-system,"Segoe UI",Roboto,Helvetica,Arial;display:flex;align-items:center;justify-content:center;background:#111;color:#fff}

    .player{width:min(900px,96vw);height:min(600px,88vh);background:#000;border-radius:18px;overflow:hidden;box-shadow:0 12px 40px rgba(0,0,0,.6);position:relative}

    .slides{width:100%;height:100%;position:relative}
    .slide{position:absolute;inset:0;display:flex;flex-direction:column;align-items:center;justify-content:center;padding:28px;background-size:cover;background-position:center;transition:opacity .8s ease,transform .8s ease;opacity:0;transform:scale(.98)}
    .slide.show{opacity:1;transform:scale(1)}

    .caption{backdrop-filter: blur(6px);background:linear-gradient(0deg,rgba(0,0,0,.5),rgba(0,0,0,.25));padding:14px 20px;border-radius:12px;max-width:88%;text-align:center}
    .caption h1{font-size:clamp(20px,4vw,36px);margin-bottom:8px}
    .caption p{font-size:clamp(12px,2.5vw,18px);opacity:.95}

    .controls{position:absolute;left:0;right:0;bottom:14px;display:flex;gap:8px;justify-content:center;align-items:center}
    .btn{background:rgba(255,255,255,.08);border:1px solid rgba(255,255,255,.08);color:#fff;padding:8px 12px;border-radius:10px;cursor:pointer;font-weight:600}
    .btn:hover{transform:translateY(-2px)}

    .top-left{position:absolute;left:16px;top:12px;background:linear-gradient(90deg,rgba(0,0,0,.45),rgba(0,0,0,.2));padding:8px 12px;border-radius:12px;font-weight:600}

    .progress{position:absolute;left:0;right:0;top:0;height:6px;background:rgba(255,255,255,.06)}
    .progress > i{display:block;height:100%;width:0%;background:linear-gradient(90deg,#ff9a9e,#fad0c4)}

    .small-note{position:absolute;right:12px;bottom:12px;font-size:12px;opacity:.8}

    /* responsive tweaks */
    @media (max-width:600px){.player{border-radius:12px;height:76vh}}
  </style>
</head>
<body>
  <div class="player" id="player">
    <div class="progress"><i id="prog"></i></div>

    <div class="slides" id="slides">
      <!-- Each .slide can use data-bg for a background image and inner caption text -->
      <div class="slide show" data-bg="https://images.unsplash.com/photo-1503264116251-35a269479413?auto=format&fit=crop&w=1600&q=80">
        <div class="caption">
          <h1 id="title">Happy Birthday, NAME!</h1>
          <p id="subtitle">Wishing you joy, love and the best year yet — here's a few memories.</p>
        </div>
      </div>

      <div class="slide" data-bg="https://images.unsplash.com/photo-1524504388940-b1c1722653e1?auto=format&fit=crop&w=1600&q=80">
        <div class="caption"><h1>Memories Together</h1><p>Remember this day — laughter, food and all the little things that made it perfect.</p></div>
      </div>

      <div class="slide" data-bg="https://images.unsplash.com/photo-1545239351-1141bd82e8a6?auto=format&fit=crop&w=1600&q=80">
        <div class="caption"><h1>Wishes</h1><p>May your dreams grow bigger and your worries grow smaller.</p></div>
      </div>

      <div class="slide" data-bg="https://images.unsplash.com/photo-1487412947147-5cebf100ffc2?auto=format&fit=crop&w=1600&q=80">
        <div class="caption"><h1>Many More</h1><p>To new adventures and another chapter of sparkle.</p></div>
      </div>

      <div class="slide" data-bg="https://images.unsplash.com/photo-1514518726906-10e66276c7b8?auto=format&fit=crop&w=1600&q=80">
        <div class="caption"><h1>With Love</h1><p>— From all of us</p></div>
      </div>

    </div>

    <div class="top-left" id="countdownText">Birthday: <span id="bdayDate">Sep 19, 2005</span></div>

    <div class="controls">
      <button class="btn" id="prev">◀ Prev</button>
      <button class="btn" id="play">Play</button>
      <button class="btn" id="next">Next ▶</button>
      <button class="btn" id="download">Download Video</button>
    </div>

    <div class="small-note">Tip: Replace images & text in code for personalization</div>
  </div>

  <!-- optional audio: replace with your hosted MP3 url or leave disabled -->
  <audio id="bgm" loop crossorigin="anonymous" preload="auto">
    <!-- Example: uncomment and paste your mp3 URL
    <source src="https://example.com/your-birthday-music.mp3" type="audio/mpeg">
    -->
  </audio>

  <script>
    // ---------- Customization ----------
    const BIRTHDAY_NAME = 'Deeksha'; // change name
    const BIRTHDAY_DATE = '2005-09-19'; // yyyy-mm-dd
    const SLIDE_DURATION = 3500; // milliseconds per slide
    // -----------------------------------

    const slidesEl = document.getElementById('slides');
    const slides = Array.from(document.querySelectorAll('.slide'));
    const title = document.getElementById('title');
    const subtitle = document.getElementById('subtitle');
    const bdayDateEl = document.getElementById('bdayDate');
    const prog = document.getElementById('prog');

    // set backgrounds from data-bg attribute
    slides.forEach(s => {
      const bg = s.dataset.bg;
      if(bg) s.style.backgroundImage = `url('${bg}')`;
    });

    // personalize text
    title.textContent = `Happy Birthday, ${BIRTHDAY_NAME}!`;
    bdayDateEl.textContent = new Date(BIRTHDAY_DATE).toLocaleDateString();

    let idx = 0, timer = null, playing = false;

    function show(i){
      slides.forEach((s,j)=> s.classList.toggle('show', j===i));
      // progress bar
      prog.style.width = `${((i+1)/slides.length)*100}%`;
    }

    function next(){ idx = (idx+1) % slides.length; show(idx); }
    function prev(){ idx = (idx-1+slides.length) % slides.length; show(idx); }

    function start(){ if(playing) return; playing = true; document.getElementById('play').textContent='Pause'; timer = setInterval(next, SLIDE_DURATION); document.getElementById('bgm').play().catch(()=>{}); }
    function stop(){ playing=false; document.getElementById('play').textContent='Play'; clearInterval(timer); document.getElementById('bgm').pause(); }

    document.getElementById('next').addEventListener('click', ()=>{ next(); if(playing){ stop(); start(); }});
    document.getElementById('prev').addEventListener('click', ()=>{ prev(); if(playing){ stop(); start(); }});
    document.getElementById('play').addEventListener('click', ()=>{ playing? stop(): start(); });

    // Download: capture the slides as a simple GIF or series of images (browser support limited)
    document.getElementById('download').addEventListener('click', async ()=>{
      alert('To make a proper video you can record the screen or use simple services like Kapwing/Canva. This button will download current slide as an image.');
      const node = slides[idx];
      const bg = node.style.backgroundImage.slice(5,-2);
      // download background only
      const a = document.createElement('a'); a.href=bg; a.download = `slide-${idx+1}.jpg`; document.body.appendChild(a); a.click(); a.remove();
    });

    // keyboard
    window.addEventListener('keydown', e=>{
      if(e.key === ' '){ e.preventDefault(); playing? stop(): start(); }
      if(e.key === 'ArrowRight') next();
      if(e.key === 'ArrowLeft') prev();
    });

    show(0);

    // Auto-play on load for small reel effect
    setTimeout(()=>{ start(); setTimeout(()=>stop(), SLIDE_DURATION * slides.length + 300); }, 500);

    // Helpful note: to fully replicate an Instagram reel you can record the player with OBS or use an online editor to add Instagram-style speed/filters.
  </script>
</body>
</html>
