<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Ordenar y clasificar números reales</title>
<style>
  :root { --bg:#f5f6fa; --card:#fff; --tx:#1f2430; --mut:#6b7280; --ac:#4f46e5; --ok:#15803d; --no:#b91c1c; --bd:#e5e7eb; }
  @media (prefers-color-scheme: dark) {
    :root { --bg:#14161c; --card:#1e2128; --tx:#e8eaf0; --mut:#9aa1ae; --ac:#818cf8; --ok:#4ade80; --no:#f87171; --bd:#2f343e; }
  }
  * { box-sizing:border-box; }
  body { margin:0; padding:24px 16px; background:var(--bg); color:var(--tx); font-family:system-ui,Segoe UI,Arial,sans-serif; }
  main { max-width:900px; margin:0 auto; }
  h1 { font-size:1.5rem; margin:0 0 4px; }
  p.sub { color:var(--mut); margin:0 0 16px; }
  .card { background:var(--card); border:1px solid var(--bd); border-radius:12px; padding:16px; margin-bottom:16px; }
  textarea { width:100%; min-height:90px; padding:10px; border:1px solid var(--bd); border-radius:8px; background:var(--bg); color:var(--tx); font:inherit; resize:vertical; }
  .row { display:flex; gap:10px; flex-wrap:wrap; align-items:center; margin-top:10px; }
  button, select { padding:9px 16px; border-radius:8px; border:1px solid var(--bd); background:var(--card); color:var(--tx); font:inherit; cursor:pointer; }
  button.main { background:var(--ac); color:#fff; border-color:var(--ac); }
  .hint { font-size:.85rem; color:var(--mut); margin-top:8px; line-height:1.5; }
  table { width:100%; border-collapse:collapse; font-size:.95rem; }
  th, td { text-align:left; padding:8px 6px; border-bottom:1px solid var(--bd); }
  th { color:var(--mut); font-weight:600; font-size:.8rem; text-transform:uppercase; }
  .tag { display:inline-block; padding:2px 8px; border-radius:999px; font-size:.8rem; border:1px solid var(--bd); }
  .si { color:var(--ok); font-weight:600; }
  .no { color:var(--no); font-weight:600; }
  .err { color:var(--no); font-size:.9rem; margin-top:8px; }
  .wrap { overflow-x:auto; }
</style>
</head>
<body>
<main>
  <h1>Ordenar y clasificar números reales</h1>
  <p class="sub">Ordena tus números y dice a qué conjunto pertenecen y si se pueden escribir de forma exacta.</p>

  <div class="card">
    <label for="entrada"><b>Escribe tus números</b> (separados por espacios, punto y coma o saltos de línea)</label>
    <textarea id="entrada">5 -3 1/3 0.75 0.(3) pi sqrt(2) sqrt(16) e -7/2 0</textarea>
    <div class="row">
      <select id="orden">
        <option value="asc">Menor a mayor</option>
        <option value="desc">Mayor a menor</option>
      </select>
      <button class="main" id="btn">Ordenar y clasificar</button>
    </div>
    <div class="hint">
      Formatos aceptados: enteros (<code>-4</code>), decimales (<code>2.5</code> o <code>2,5</code>), fracciones (<code>3/7</code>),
      decimales periódicos (<code>0.(3)</code>, <code>1.2(45)</code>), <code>pi</code>, <code>e</code>, <code>phi</code>
      y raíces (<code>sqrt(2)</code> o <code>√2</code>).
    </div>
    <div class="err" id="err"></div>
  </div>

  <div class="card" id="resCard" style="display:none">
    <div class="wrap">
      <table>
        <thead>
          <tr><th>#</th><th>Número</th><th>Valor aprox.</th><th>Conjunto</th><th>¿Se puede escribir exacto?</th></tr>
          </thead>
        <tbody id="tbody"></tbody>
      </table>
    </div>
    <div class="hint">
      <b>Conjuntos:</b> ℕ naturales ⊂ ℤ enteros ⊂ ℚ racionales (fracciones, decimales finitos o periódicos). Los irracionales (𝕀) tienen infinitos decimales sin patrón:
      no se pueden escribir completos, solo aproximar. ℚ ∪ 𝕀 = ℝ.
    </div>
  </div>
</main>

<script>
function gcd(a, b) { a = Math.abs(a); b = Math.abs(b); while (b) { [a, b] = [b, a % b]; } return a; }

function esCuadrado(n) { const r = Math.round(Math.sqrt(n)); return r * r === n ? r : null; }

// Convierte un decimal finito (string positivo) en [num, den]
function decimalAFraccion(s) {
  const [ent, dec = ""] = s.split(".");
  const den = Math.pow(10, dec.length);
  return [parseInt(ent + dec, 10), den];
}

function crearRacional(n, d, texto) {
  if (d < 0) { n = -n; d = -d; }
  const g = gcd(n, d) || 1;
  n /= g; d /= g;
  let tipo = "rat";
  if (d === 1) tipo = n >= 0 ? "nat" : "int";
  return { texto, valor: n / d, tipo, frac: [n, d] };
}

function interpretar(tok) {
  const t = tok.trim().toLowerCase().replace(",", ".").replace("π", "pi").replace("φ", "phi");
  let m;

  if (t === "pi") return { texto: "π", valor: Math.PI, tipo: "irr", frac: null };
  if (t === "e") return { texto: "e", valor: Math.E, tipo: "irr", frac: null };
  if (t === "phi") return { texto: "φ", valor: (1 + Math.sqrt(5)) / 2, tipo: "irr", frac: null };

  // raíz cuadrada
  if ((m = t.match(/^(?:sqrt|raiz|√)\(?(\d+(?:\.\d+)?)\)?$/))) {
    const x = m[1];
    const [n, d] = decimalAFraccion(x);
    const rn = esCuadrado(n), rd = esCuadrado(d);
    if (rn !== null && rd !== null) return crearRacional(rn, rd, "√" + x);
    return { texto: "√" + x, valor: Math.sqrt(parseFloat(x)), tipo: "irr", frac: null };
  }

  // fracción a/b
  if ((m = t.match(/^(-?\d+)\/(-?\d+)$/))) {
    const n = parseInt(m[1], 10), d = parseInt(m[2], 10);
    if (d === 0) throw new Error("División entre cero en «" + tok + "»");
    return crearRacional(n, d, tok);
  }

  // decimal periódico: 0.(3) o 1.2(45)
  if ((m = t.match(/^(-?)(\d+)\.(\d*)\((\d+)\)$/))) {
    const signo = m[1] ? -1 : 1;
    const ent = m[2], noPer = m[3], per = m[4];
    const A = parseInt(ent + noPer + per, 10);
    const B = parseInt(ent + noPer, 10);
    const den = Math.pow(10, noPer.length) * (Math.pow(10, per.length) - 1);
    return crearRacional(signo * (A - B), den, tok);
  }

  // entero o decimal finito
  if ((m = t.match(/^(-?)(\d+(?:\.\d+)?)$/))) {
    const signo = m[1] ? -1 : 1;
    const [n, d] = decimalAFraccion(m[2]);
    return crearRacional(signo * n, d, tok);
  }

  throw new Error("No entiendo «" + tok + "»");
}

const NOMBRES = {
  nat: "ℕ natural (también ℤ, ℚ, ℝ)",
  int: "ℤ entero (también ℚ, ℝ)",
  rat: "ℚ racional (también ℝ)",
  irr: "𝕀 irracional (también ℝ)"
};

function fmt(v) {
  if (Number.isInteger(v)) return String(v);
  return v.toPrecision(10).replace(/0+$/, "").replace(/\.$/, "");
}

function procesar() {
  const err = document.getElementById("err");
  err.textContent = "";
  const tokens = document.getElementById("entrada").value.split(/[;\s]+/).filter(Boolean);
  const lista = [];
  const errores = [];

  for (const tk of tokens) {
    try { lista.push(interpretar(tk)); }
    catch (e) { errores.push(e.message); }
  }
  if (errores.length) err.textContent = errores.join(" · ");

  const dir = document.getElementById("orden").value === "asc" ? 1 : -1;
  lista.sort((a, b) => dir * (a.valor - b.valor));

  const tbody = document.getElementById("tbody");
  tbody.innerHTML = "";
  lista.forEach((x, i) => {
    const tr = document.createElement("tr");
    let escritura;
    if (x.tipo === "irr") {
      escritura = '<span class="no">No</span> — solo aproximado';
    } else if (x.tipo === "rat") {
      escritura = '<span class="si">Sí</span> — ' + x.frac[0] + "/" + x.frac[1];
    } else {
      escritura = '<span class="si">Sí</span> — ' + x.frac[0];
    }
    tr.innerHTML =
      "<td>" + (i + 1) + "</td>" +
      "<td><b>" + x.texto + "</b></td>" +
      "<td>" + (x.tipo === "irr" ? "≈ " : "") + fmt(x.valor) + "</td>" +
      '<td><span class="tag">' + NOMBRES[x.tipo] + "</span></td>" +
      "<td>" + escritura + "</td>";
    tbody.appendChild(tr);
  });

  document.getElementById("resCard").style.display = lista.length ? "block" : "none";
}

document.getElementById("btn").addEventListener("click", procesar);
document.getElementById("orden").addEventListener("change", procesar);
procesar();
</script>
</body>
</html>
