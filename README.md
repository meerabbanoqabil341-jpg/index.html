# index.html
cooking recipes
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Home Cook — Recipes & Tips</title>
  <meta name="description" content="A simple responsive cooking website template with recipe cards, search, filters and recipe detail modal. Ready for Netlify or GitHub Pages." />
  <!-- Simple modern CSS (no external libs) -->
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--muted:#9aa4b2;--accent:#ff7a59;--glass: rgba(255,255,255,0.04)}
    *{box-sizing:border-box}
    body{margin:0;font-family:Inter,ui-sans-serif,system-ui,-apple-system,"Segoe UI",Roboto,"Helvetica Neue",Arial;color:#e6eef6;background:linear-gradient(180deg,#071124 0%, #091826 100%);-webkit-font-smoothing:antialiased}
    .wrap{max-width:1100px;margin:36px auto;padding:20px}

    header{display:flex;align-items:center;justify-content:space-between;gap:16px}
    .brand{display:flex;align-items:center;gap:12px}
    .logo{width:48px;height:48px;border-radius:8px;background:linear-gradient(135deg,var(--accent),#ffd39f);display:flex;align-items:center;justify-content:center;font-weight:700;color:#1b1b1b}
    h1{font-size:20px;margin:0}
    p.lead{margin:0;color:var(--muted);font-size:14px}

    .controls{display:flex;gap:12px;flex-wrap:wrap;margin-top:18px}
    .search{flex:1;min-width:180px}
    input[type="search"]{width:100%;padding:10px 14px;border-radius:10px;border:1px solid rgba(255,255,255,0.06);background:var(--glass);color:inherit}
    .select, .btn{padding:10px 14px;border-radius:10px;border:1px solid rgba(255,255,255,0.06);background:transparent;color:inherit}
    .btn{cursor:pointer;background:linear-gradient(90deg,var(--accent),#ffaf84);border:none;color:#111}

    .grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(240px,1fr));gap:18px;margin-top:22px}
    .card{background:linear-gradient(180deg, rgba(255,255,255,0.02), rgba(255,255,255,0.01));border-radius:12px;overflow:hidden;border:1px solid rgba(255,255,255,0.03);box-shadow:0 6px 18px rgba(2,6,23,0.6)}
    .card img{width:100%;height:150px;object-fit:cover;display:block}
    .card-content{padding:12px}
    .card h3{margin:0 0 6px 0;font-size:16px}
    .meta{display:flex;align-items:center;gap:8px;color:var(--muted);font-size:13px}
    .tags{margin-top:10px;display:flex;gap:8px;flex-wrap:wrap}
    .tag{background:rgba(255,255,255,0.03);padding:6px 8px;border-radius:999px;font-size:12px;color:var(--muted)}

    footer{margin-top:28px;color:var(--muted);font-size:13px;text-align:center}

    /* modal */
    .modal-backdrop{position:fixed;inset:0;background:linear-gradient(180deg,rgba(2,6,23,0.6),rgba(2,6,23,0.8));display:none;align-items:center;justify-content:center;padding:24px}
    .modal{max-width:920px;width:100%;background:linear-gradient(180deg,#071829,#05121b);border-radius:12px;overflow:hidden;border:1px solid rgba(255,255,255,0.04);display:grid;grid-template-columns:1fr 360px;gap:0}
    .modal img{width:100%;height:100%;object-fit:cover}
    .modal-content{padding:18px}
    .close{background:transparent;border:0;color:var(--muted);font-weight:700;cursor:pointer}

    /* responsive */
    @media (max-width:880px){.modal{grid-template-columns:1fr}} 
    @media (max-width:520px){.brand h1{font-size:18px}.card img{height:140px}}

    /* small util */
    .hidden{display:none}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <div class="brand">
        <div class="logo">HC</div>
        <div>
          <h1>Home Cook</h1>
          <p class="lead">Simple recipes, quick tips — cook with heart.</p>
        </div>
      </div>
      <nav>
        <button class="btn" id="addRecipeBtn">+ Add Recipe</button>
      </nav>
    </header>

    <div class="controls">
      <div class="search">
        <input id="search" type="search" placeholder="Search recipes, ingredients..." />
      </div>
      <select id="category" class="select">
        <option value="all">All categories</option>
        <option value="breakfast">Breakfast</option>
        <option value="lunch">Lunch</option>
        <option value="dinner">Dinner</option>
        <option value="dessert">Dessert</option>
        <option value="vegan">Vegan</option>
      </select>
      <select id="sort" class="select">
        <option value="popular">Sort: Popular</option>
        <option value="newest">Newest</option>
        <option value="time">Shortest time</option>
      </select>
    </div>

    <main>
      <section class="grid" id="recipesGrid" aria-live="polite">
        <!-- recipe cards injected by JS -->
      </section>
    </main>

    <footer>
      Built for Netlify / GitHub Pages — customize & deploy. © <span id="year"></span>
    </footer>
  </div>

  <!-- Modal (recipe detail) -->
  <div id="modalBackdrop" class="modal-backdrop" role="dialog" aria-hidden="true">
    <div class="modal" role="document">
      <div id="modalImageWrap">
        <img id="modalImage" src="" alt="Recipe image" />
      </div>
      <div class="modal-content">
        <div style="display:flex;justify-content:space-between;align-items:start">
          <div>
            <h2 id="modalTitle">Recipe Title</h2>
            <div class="meta"><span id="modalTime">--</span> • <span id="modalServings">--</span> servings</div>
          </div>
          <button class="close" id="closeModal">✕</button>
        </div>
        <h4>Ingredients</h4>
        <ul id="modalIngredients"></ul>
        <h4>Instructions</h4>
        <ol id="modalInstructions"></ol>
      </div>
    </div>
  </div>

  <!-- Minimal JS: data + interactivity -->
  <script>
    // sample recipe data (you can replace with a JSON file or API)
    const recipes = [
      {
        id:1,
        title: 'Masala Omelette',
        category: 'breakfast',
        time: 10,
        servings:1,
        popular: 95,
        img: 'https://images.unsplash.com/photo-1550304943-7e0f34d3f6d7?q=80&w=1200&auto=format&fit=crop&ixlib=rb-4.0.3&s=0',
        ingredients: ['2 eggs','1 small onion (finely chopped)','1 green chilli','Salt & pepper','1 tsp oil'],
        instructions: ['Beat eggs with salt & pepper','Saute onion & chilli in oil','Pour eggs and cook both sides','Serve hot with bread']
      },
      {
        id:2,
        title: 'Quick Chana Chaat',
        category: 'lunch',
        time: 15,
        servings:2,
        popular: 88,
        img: 'https://images.unsplash.com/photo-1604908177522-0b257b2afd4d?q=80&w=1200&auto=format&fit=crop&ixlib=rb-4.0.3&s=0',
        ingredients: ['1 cup boiled chickpeas','1 tomato chopped','1/2 onion','Chaat masala & lemon','Coriander'],
        instructions: ['Mix all ingredients','Sprinkle chaat masala & lemon','Garnish with coriander','Serve chilled']
      },
      {
        id:3,
        title: 'Creamy Pasta Alfredo',
        category: 'dinner',
        time: 25,
        servings:2,
        popular: 78,
        img: 'https://images.unsplash.com/photo-1601315578319-6b4b9f6f8a6b?q=80&w=1200&auto=format&fit=crop&ixlib=rb-4.0.3&s=0',
        ingredients: ['200g pasta','1 cup cream','1/2 cup parmesan','2 cloves garlic','Salt & pepper'],
        instructions: ['Cook pasta al dente','Saute garlic, add cream & cheese','Toss pasta in sauce','Serve with extra parmesan']
      },
      {
        id:4,
        title: 'Chocolate Mug Cake',
        category: 'dessert',
        time: 5,
        servings:1,
        popular: 85,
        img: 'https://images.unsplash.com/photo-1599785209707-0f6f185d4b07?q=80&w=1200&auto=format&fit=crop&ixlib=rb-4.0.3&s=0',
        ingredients: ['4 tbsp flour','2 tbsp cocoa powder','2 tbsp sugar','3 tbsp milk','1 tbsp oil'],
        instructions: ['Mix dry ingredients','Add milk & oil','Microwave 90 seconds','Let cool slightly and eat']
      }
    ];

    // DOM refs
    const grid = document.getElementById('recipesGrid');
    const searchEl = document.getElementById('search');
    const categoryEl = document.getElementById('category');
    const sortEl = document.getElementById('sort');
    const yearSpan = document.getElementById('year');

    // modal refs
    const modalBackdrop = document.getElementById('modalBackdrop');
    const modalTitle = document.getElementById('modalTitle');
    const modalImage = document.getElementById('modalImage');
    const modalTime = document.getElementById('modalTime');
    const modalServings = document.getElementById('modalServings');
    const modalIngredients = document.getElementById('modalIngredients');
    const modalInstructions = document.getElementById('modalInstructions');
    const closeModalBtn = document.getElementById('closeModal');

    // helpers
    function renderCard(r){
      const el = document.createElement('article');
      el.className = 'card';
      el.innerHTML = `
        <img loading="lazy" src="${r.img}" alt="${r.title}" />
        <div class="card-content">
          <h3>${r.title}</h3>
          <div class="meta">⏱ ${r.time} min • 🍽 ${r.servings}</div>
          <div class="tags">
            <span class="tag">${r.category}</span>
            <span class="tag">${Math.round(r.popular)}% liked</span>
          </div>
        </div>
      `;
      el.addEventListener('click', ()=> openModal(r.id));
      return el;
    }

    function openModal(id){
      const r = recipes.find(x=>x.id===id);
      if(!r) return;
      modalTitle.textContent = r.title;
      modalImage.src = r.img;
      modalTime.textContent = r.time + ' min';
      modalServings.textContent = r.servings;
      modalIngredients.innerHTML = r.ingredients.map(i=>`<li>${i}</li>`).join('');
      modalInstructions.innerHTML = r.instructions.map(s=>`<li>${s}</li>`).join('');
      modalBackdrop.style.display = 'flex';
      modalBackdrop.setAttribute('aria-hidden','false');
    }

    function closeModal(){
      modalBackdrop.style.display = 'none';
      modalBackdrop.setAttribute('aria-hidden','true');
    }

    closeModalBtn.addEventListener('click', closeModal);
    modalBackdrop.addEventListener('click', (e)=>{ if(e.target===modalBackdrop) closeModal(); });

    // render grid
    function renderGrid(data){
      grid.innerHTML = '';
      if(data.length===0){
        grid.innerHTML = '<p style="color:var(--muted)">No recipes found — try a different search.</p>';
        return;
      }
      data.forEach(r=> grid.appendChild(renderCard(r)));
    }

    // filtering
    function applyFilters(){
      const q = searchEl.value.trim().toLowerCase();
      const cat = categoryEl.value;
      const sort = sortEl.value;
      let out = recipes.filter(r=>{
        const matchesQ = q === '' || (
          r.title.toLowerCase().includes(q) ||
          r.ingredients.join(' ').toLowerCase().includes(q) ||
          r.category.toLowerCase().includes(q)
        );
        const matchesCat = cat === 'all' || r.category === cat;
        return matchesQ && matchesCat;
      });

      if(sort === 'popular') out.sort((a,b)=>b.popular - a.popular);
      if(sort === 'newest') out.sort((a,b)=>b.id - a.id);
      if(sort === 'time') out.sort((a,b)=>a.time - b.time);

      renderGrid(out);
    }

    // init
    (function(){
      yearSpan.textContent = new Date().getFullYear();
      renderGrid(recipes);
      searchEl.addEventListener('input', debounce(applyFilters, 250));
      categoryEl.addEventListener('change', applyFilters);
      sortEl.addEventListener('change', applyFilters);

      // Add recipe button placeholder behaviour
      document.getElementById('addRecipeBtn').addEventListener('click', ()=>{
        alert('To add recipes, edit the recipes array in the HTML file or connect to a simple JSON file / API.');
      });
    })();

    // utility: debounce
    function debounce(fn, wait){
      let t;
      return function(...args){
        clearTimeout(t);
        t = setTimeout(()=>fn.apply(this,args), wait);
      }
    }
  </script>

</body>
</html>

