
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>LogísticaTú</title>
  <meta name="description" content="Portal de gestión de envíos y flota – LogísticaTú🇵🇾" />

  <!-- Leaflet CSS -->
  <link rel="stylesheet" href="https://unpkg.com/leaflet@1.9.4/dist/leaflet.css" />
  <style>
    :root {
      --bg: #ffffff;
      --bg-alt: #f5f7fa;
      --text: #111827;
      --accent: #0b9e6f;
      --radius: .75rem;
    }

    @media (prefers-color-scheme: dark) {
      :root {
        --bg: #111827;
        --bg-alt: #1f2937;
        --text: #f9fafb;
        --accent: #0bd890;
      }
    }

    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: system-ui, sans-serif;
      background: var(--bg);
      color: var(--text);
      line-height: 1.6;
      display: grid;
      grid-template-rows: auto 1fr auto;
      min-height: 100vh;
    }

    header, footer { padding: 1rem 2rem; }

    header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      background: var(--bg-alt);
      border-bottom: 1px solid #00000011;
    }

    header h1 { font-size: 1.5rem; }

    #searchToggle { cursor: pointer; background: none; border: 0; font-size: 1.5rem; }

    #searchBar {
      display: none;
      width: 100%;
      padding: .5rem;
      margin-top: .5rem;
      border: 1px solid var(--accent);
      border-radius: var(--radius);
    }

    main {
      padding: 2rem;
      display: grid;
      gap: 2rem;
      grid-template-columns: 1fr 1fr;
    }

    section {
      background: var(--bg-alt);
      padding: 1.5rem;
      border-radius: var(--radius);
      box-shadow:  0 2px 6px #0000001a;
    }

    h2 { margin-bottom: 1rem; }

    label { display: block; margin-top: .75rem; font-weight: 600; }

    input, select, textarea {
      width: 100%;
      padding: .5rem;
      margin-top: .25rem;
      border: 1px solid var(--accent);
      border-radius: var(--radius);
      background: var(--bg);
      color: var(--text);
    }

    button[type="submit"] {
      margin-top: 1rem;
      padding: .6rem 1.2rem;
      background: var(--accent);
      border: 0;
      border-radius: var(--radius);
      color: #fff;
      font-weight: 600;
      cursor: pointer;
    }

    button[type="submit"]:hover { opacity: .9; }

    #precioEstimado {
      margin-top: .5rem;
      font-weight: 700;
    }

    #map {
      height: 400px;
      border-radius: var(--radius);
    }

    footer {
      text-align: center;
      font-size: .875rem;
      opacity: .7;
    }

    @media (max-width: 900px) {
      main { grid-template-columns: 1fr; }
    }
  </style>
</head>

<body>
  <header>
    <h1>LogísticaTú</h1>
    <button id="searchToggle" aria-label="Buscar" title="Buscar">🔍</button>
  </header>

  <input id="searchBar" type="search" placeholder="Buscar envíos, clientes, conductores…" />

  <main>
    <section>
      <h2>Alta de nuevo envío</h2>
      <form id="altaForm">
        <label>Cliente
          <input type="text" name="cliente" required />
        </label>

        <label>Dirección destino
          <input type="text" id="direccion" name="direccion" placeholder="Calle Nº, Ciudad, Paraguay" required />
        </label>

        <label>Peso (kg)
          <input type="number" name="peso" min="0.1" step="0.1" required />
        </label>

        <label>Tipo de servicio
          <select name="servicio" required>
            <option value="estandar">Estándar</option>
            <option value="express">Express</option>
            <option value="fragil">Frágil</option>
          </select>
        </label>

        <label>Observaciones
          <textarea name="obs" rows="2"></textarea>
        </label>

        <p id="precioEstimado">Precio estimado: ₲0</p>

        <button type="submit">Registrar envío</button>
      </form>
    </section>

    <section>
      <h2>Ubicación del destino – Paraguay</h2>
      <div id="map"></div>
    </section>
  </main>

  <footer>
    &copy; 2025 LogísticaTú – Derechos Reservados a Marcelo Benitez Noguera | HTML5
  </footer>

  <!-- Leaflet JS -->
  <script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>

  <script>
    const TASA_USD_A_PYG = 7817;

    /* Buscador */
    const btn = document.getElementById('searchToggle');
    const bar = document.getElementById('searchBar');
    btn.addEventListener('click', () => {
      bar.classList.toggle('show');
      bar.focus();
    });
    document.addEventListener('keydown', e => {
      if (e.key === 'Escape') bar.classList.remove('show');
    });

    /* Precio en guaraníes */
    const form = document.getElementById('altaForm');
    const precioOut = document.getElementById('precioEstimado');

    function calcularPrecio() {
      const peso = parseFloat(form.peso.value) ||  0;
      const servicio = form.servicio.value;
      const tarifasUSD = { estandar: 2, express: 4, fragil: 5 };
      const precioPYG = peso * tarifasUSD[servicio] * TASA_USD_A_PYG;
      precioOut.textContent = `Precio estimado: ₲${precioPYG.toLocaleString('es-PY')}`;
    }

    form.querySelectorAll('input, select').forEach(el => {
      el.addEventListener('input', calcularPrecio);
    });

    /* Mapa Paraguay */
    const map = L.map('map').setView([-25.2637, -57.5759], 7);
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; OpenStreetMap contributors'
    }).addTo(map);

    let marker = L.marker([-25.2637, -57.5759]).addTo(map)
                  .bindPopup('Arrastra para ajustar').openPopup();

    const direccionInput = document.getElementById('direccion');
    direccionInput.addEventListener('change', () => {
      const query = encodeURIComponent(direccionInput.value + ', Paraguay');
      fetch(`https://nominatim.openstreetmap.org/search?format=json&q=${query}`)
        .then(res => res.json())
        .then(data => {
          if (data && data[0]) {
            const { lat, lon } = data[0];
            map.setView([lat, lon], 15);
            marker.setLatLng([lat, lon]).bindPopup(direccionInput.value).openPopup();
          }
        })
        .catch(() => alert('No se pudo geolocalizar la dirección'));
    });

    form.addEventListener('submit', e => {
      e.preventDefault();
      alert('Envío registrado correctamente en LogísticaTú');
      form.reset();
      calcularPrecio();
      marker.closePopup();
    });
  </script>
</body>
</html>

