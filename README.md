<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Simulador Liga 1 Perú</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: #121212;
      color: #eee;
      padding: 20px;
      margin: 0;
    }
    
    h1, h2 {
      text-align: center;
      margin-bottom: 20px;
      font-weight: 700;
      color: #ffd700;
      text-shadow: 1px 1px 5px #444;
    }
    
    .controls {
      text-align: center;
      margin-bottom: 25px;
    }
    
    select {
      padding: 8px 12px;
      margin: 0 10px;
      border-radius: 4px;
      border: none;
      font-weight: 600;
      cursor: pointer;
      background-color: #222;
      color: #eee;
      box-shadow: 0 0 5px #444;
      transition: background-color 0.3s;
    }
    
    select:hover {
      background-color: #333;
    }
    
    #fixture {
      max-width: 600px;
      margin: 0 auto 40px auto;
      background-color: #1e1e1e;
      padding: 15px 20px;
      border-radius: 8px;
      box-shadow: 0 0 10px #222;
      font-size: 18px;
      color: #ddd;
    }
    
    #fixture div {
      margin: 9px 0;
      text-align: center;
    }
    
    #fixture input[type=number] {
      width: 45px;
      padding: 5px 0;
      text-align: center;
      border-radius: 4px;
      border: 1px solid #444;
      background-color: #121212;
      color: #eee;
      font-weight: bold;
      margin: 0 7px;
      box-shadow: inset 0 0 5px #000;
      transition: border-color 0.3s;
    }
    
    #fixture input[type=number]:focus {
      border-color: #ffd700;
      outline: none;
    }
    
    table {
      width: 90%;
      max-width: 700px;
      margin: 0 auto 50px auto;
      border-collapse: collapse;
      font-size: 16px;
      box-shadow: 0 0 15px #222;
      border-radius: 8px;
      overflow: hidden;
      background-color: #1f1f1f;
    }
    
    thead {
      background: #333;
    }
    
    thead th {
      color: #ffd700;
      font-weight: 700;
      padding: 10px 6px;
      border-bottom: 2px solid #444;
    }
    
    tbody tr:nth-child(even) {
      background-color: #222;
    }
    
    tbody tr:hover {
      background-color: #333;
      color: #ffd700;
      font-weight: 600;
      cursor: default;
    }
    
    tbody td {
      padding: 8px 6px;
      border-bottom: 1px solid #444;
      color: #ddd;
      font-weight: 600;
    }
    
    /* Zonas colores */
    .libertadores { background-color: #1e7e34; }
    .sudamericana { background-color: #b8860b; }
    .descenso { background-color: #8b0000; }
  </style>
</head>
<body>
  <h1>Simulador Liga 1 Perú</h1>

  <div class="controls">
    <select id="torneoSelect">
      <option value="apertura">Apertura</option>
      <option value="clausura">Clausura</option>
    </select>

    <select id="fechaSelect"></select>
  </div>

  <div id="fixture"></div>

  <h2>Tabla de Posiciones</h2>

  <table id="tabla">
    <thead>
      <tr>
        <th>#</th>
        <th>Equipo</th>
        <th>PJ</th>
        <th>PTS</th>
        <th>GF</th>
        <th>GC</th>
        <th>DG</th>
      </tr>
    </thead>
    <tbody></tbody>
  </table>

  <script>
    const equipos = [
      "ADT",
      "Alianza Atlético",
      "Alianza Lima",
      "Atlético Grau",
      "Cienciano",
      "Comerciantes Unidos",
      "Cusco FC",
      "Deportivo Garcilaso",
      "Deportivo Moquegua",
      "FBC Melgar",
      "UTC",
      "Juan Pablo II",
      "Los Chankas",
      "Sport Boys",
      "Sport Huancayo",
      "Sporting Cristal",
      "Universitario",
      "Unión Comercio"
    ];

    let fixtureApertura = [];
    let fixtureClausura = [];
    let tabla = {};
    let torneoActual = "apertura";

    // ===== GENERADOR ROUND ROBIN =====
    function generarFixtureBase() {
      let teams = [...equipos];
      let jornadas = teams.length - 1;
      let mitad = teams.length / 2;
      let fixture = [];

      for (let j = 0; j < jornadas; j++) {
        let fecha = [];

        for (let i = 0; i < mitad; i++) {
          fecha.push({
            local: teams[i],
            visita: teams[teams.length - 1 - i],
            gl: null,
            gv: null
          });
        }

        fixture.push(fecha);
        teams.splice(1, 0, teams.pop());
      }

      return fixture;
    }

    function generarClausura(apertura) {
      return apertura.map(fecha =>
        fecha.map(p => ({
          local: p.visita,
          visita: p.local,
          gl: null,
          gv: null
        }))
      );
    }

    // ===== RENDER FECHAS =====
    function renderFechas() {
      const fechaSelect = document.getElementById("fechaSelect");
      fechaSelect.innerHTML = "";

      let fixture = torneoActual === "apertura"
        ? fixtureApertura
        : fixtureClausura;

      fixture.forEach((_, i) => {
        let option = document.createElement("option");
        option.value = i;
        option.textContent = "Fecha " + (i + 1);
        fechaSelect.appendChild(option);
      });
    }

    // ===== RENDER FIXTURE =====
    function renderFixture(fechaIndex) {
      const div = document.getElementById("fixture");
      div.innerHTML = "";

      let fixture = torneoActual === "apertura"
        ? fixtureApertura
        : fixtureClausura;

      fixture[fechaIndex].forEach((p, index) => {
        div.innerHTML += `
          <div>
            ${p.local}
            <input type="number" min="0" value="${p.gl ?? ""}"
              onchange="actualizar(${fechaIndex},${index},this.value,'local')">
            -
            <input type="number" min="0" value="${p.gv ?? ""}"
              onchange="actualizar(${fechaIndex},${index},this.value,'visita')">
            ${p.visita}
          </div>
        `;
      });
    }

    // ===== ACTUALIZAR =====
    function actualizar(fecha, partido, valor, tipo) {
      let fixture = torneoActual === "apertura"
        ? fixtureApertura
        : fixtureClausura;

      if (tipo === "local") {
        fixture[fecha][partido].gl = parseInt(valor);
      } else {
        fixture[fecha][partido].gv = parseInt(valor);
      }

      calcularTabla();
    }

    // ===== CALCULAR TABLA =====
    function calcularTabla() {
      tabla = {};

      equipos.forEach(e => {
        tabla[e] = { pts: 0, gf: 0, gc: 0, pj: 0 };
      });

      let fixture = torneoActual === "apertura"
        ? fixtureApertura
        : fixtureClausura;

      fixture.forEach(fecha => {
        fecha.forEach(p => {
          if (p.gl !== null && p.gv !== null) {
            tabla[p.local].pj++;
            tabla[p.visita].pj++;

            tabla[p.local].gf += p.gl;
            tabla[p.local].gc += p.gv;

            tabla[p.visita].gf += p.gv;
            tabla[p.visita].gc += p.gl;

            if (p.gl > p.gv) tabla[p.local].pts += 3;
            else if (p.gl < p.gv) tabla[p.visita].pts += 3;
            else {
              tabla[p.local].pts += 1;
              tabla[p.visita].pts += 1;
            }
          }
        });
      });

      renderTabla();
    }

    // ===== RENDER TABLA =====
    function renderTabla() {
      let tbody = document.querySelector("#tabla tbody");
      tbody.innerHTML = "";

      let orden = Object.entries(tabla).sort((a, b) =>
        b[1].pts - a[1].pts ||
        (b[1].gf - b[1].gc) - (a[1].gf - a[1].gc) ||
        b[1].gf - a[1].gf
      );

      orden.forEach((equipo, i) => {
        let dg = equipo[1].gf - equipo[1].gc;

        tbody.innerHTML += `
          <tr>
            <td>${i + 1}</td>
            <td>${equipo[0]}</td>
            <td>${equipo[1].pj}</td>
            <td>${equipo[1].pts}</td>
            <td>${equipo[1].gf}</td>
            <td>${equipo[1].gc}</td>
            <td>${dg}</td>
          </tr>
        `;
      });
    }

    // ===== EVENTOS =====
    document.getElementById("fechaSelect").addEventListener("change", e => {
      renderFixture(e.target.value);
    });

    document.getElementById("torneoSelect").addEventListener("change", e => {
      torneoActual = e.target.value;
      renderFechas();
      renderFixture(0);
      calcularTabla();
    });

    // ===== INICIO =====
    fixtureApertura = generarFixtureBase();
    fixtureClausura = generarClausura(fixtureApertura);

    renderFechas();
    renderFixture(0);
    calcularTabla();
  </script>
</body>
</html>
