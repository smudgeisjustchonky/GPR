<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<title>Proportional Relationship Grapher</title>

<!-- Plotly.js from CDN -->
<script src="https://cdn.plot.ly/plotly-2.35.2.min.js"></script>

<style>
  body {font-family: Arial, Helvetica, sans-serif; margin: 20px; background:#f9f9f9;}
  .container {max-width: 800px; margin:auto;}
  .input-group {margin-bottom:12px;}
  label {font-weight: bold; display:block; margin-bottom:4px;}
  input[type=text] {width:100%; padding:8px; font-size:1rem;}
  button {padding:8px 16px; font-size:1rem; margin-top:8px;}
  #graph {width:100%; height:500px; margin-top:20px;}
  #ollama-response {margin-top:20px; padding:10px; background:#e8f5e9; border:1px solid #c8e6c9; white-space:pre-wrap;}
</style>
</head>
<body>
<div class="container">
  <h1>Proportional Relationship Grapher</h1>

  <div class="input-group">
    <label for="ratio">Enter a ratio (e.g. <code>3:5</code> or <code>7/2</code> or <code>1.5</code>)</label>
    <input id="ratio" type="text" placeholder="3:5">
  </div>
  <button id="drawBtn">Draw Graph</button>

  <div id="graph"></div>

  <hr>

  <h2>Ask Ollama (optional)</h2>
  <div class="input-group">
    <label for="ollamaPrompt">Prompt for Ollama (e.g., “Explain the ratio 3:5”)</label>
    <input id="ollamaPrompt" type="text" placeholder="Explain the ratio 3:5">
  </div>
  <button id="askOllamaBtn">Ask Ollama</button>

  <div id="ollama-response"></div>
</div>

<script>
/* ------------------------------------------------------------------ */
/*  Utility: turn a user‑entered ratio string into a numeric constant k  */
/* ------------------------------------------------------------------ */
function parseRatio(str) {
  // Trim whitespace and remove surrounding parentheses
  str = str.trim().replace(/^[(\[]|[)\]]$/g, '');

  // 1) "a:b"   →  k = b / a
  const colonMatch = str.match(/^([-+]?\d*\.?\d+)\s*[:]\s*([-+]?\d*\.?\d+)$/);
  if (colonMatch) {
    const a = parseFloat(colonMatch[1]);
    const b = parseFloat(colonMatch[2]);
    if (a === 0) throw new Error('First number of the ratio cannot be zero.');
    return b / a;
  }

  // 2) "a/b"   →  k = a / b   (some people write ratio as a/b)
  const slashMatch = str.match(/^([-+]?\d*\.?\d+)\s*[\/]\s*([-+]?\d*\.?\d+)$/);
  if (slashMatch) {
    const a = parseFloat(slashMatch[1]);
    const b = parseFloat(slashMatch[2]);
    if (b === 0) throw new Error('Denominator cannot be zero.');
    return a / b;
  }

  // 3) single number → treat it as k directly
  const numberMatch = str.match(/^[-+]?\d*\.?\d+$/);
  if (numberMatch) return parseFloat(str);

  throw new Error('Could not understand the ratio. Use "a:b", "a/b", or a single number.');
}

/* ------------------------------------------------------------------ */
/*  Plotting logic                                                    */
/* ------------------------------------------------------------------ */
function drawGraph(k) {
  const x = [];
  const y = [];
  // We'll plot from -10 to 10 (skip zero if you want)
  for (let i = -10; i <= 10; i += 0.1) {
    x.push(i);
    y.push(k * i);
  }

  const trace = {
    x: x,
    y: y,
    mode: 'lines',
    name: `y = ${k.toFixed(4)}·x`,
    line: {color: '#1976d2', width: 2}
  };

  const layout = {
    title: `Proportional line (k = ${k.toFixed(4)})`,
    xaxis: {title: 'x', zeroline: true},
    yaxis: {title: 'y', zeroline: true},
    showlegend: true,
    hovermode: 'closest'
  };

  Plotly.newPlot('graph', [trace], layout);
}

/* ------------------------------------------------------------------ */
/*  UI event handlers                                                 */
/* ------------------------------------------------------------------ */
document.getElementById('drawBtn').addEventListener('click', () => {
  const input = document.getElementById('ratio').value;
  try {
    const k = parseRatio(input);
    drawGraph(k);
  } catch (e) {
    alert('❗️ ' + e.message);
  }
});

/* ------------------------------------------------------------------ */
/*  Ollama integration (optional)                                     */
/* ------------------------------------------------------------------ */
async function askOllama(prompt) {
  const url = 'http://127.0.0.1:11434/api/chat';

  const body = {
    model: 'gpt-oss:120b-cloud',
    messages: [
      // Add system message to control response style
      {
        role: 'system',
        content: 'Always give very short, simple responses. Express all fractions as -/- or in decimal form for easy to convert fractions. Keep explanations under 2 sentences.'
      },
      {role: 'user', content: prompt}
    ],
    stream: false
  };

  const response = await fetch(url, {
    method: 'POST',
    headers: {'Content-Type': 'application/json'},
    body: JSON.stringify(body)
  });

  if (!response.ok) {
    throw new Error(`Ollama returned ${response.status}`);
  }

  const data = await response.json();
  return data.message?.content ?? JSON.stringify(data);
}

document.getElementById('askOllamaBtn').addEventListener('click', async () => {
  const prompt = document.getElementById('ollamaPrompt').value.trim();
  const outDiv = document.getElementById('ollama-response');
  if (!prompt) {
    outDiv.textContent = 'u gonna ask me something or what?';
    return;
  }
  outDiv.textContent = 'hold up im thinking...';

  try {
    const answer = await askOllama(prompt);
    outDiv.textContent = answer;
  } catch (err) {
    outDiv.textContent = `MY BRAIN AHHH TO MUCH: ${err.message}`;
  }
});

/* ------------------------------------------------------------------ */
/*  Initial graph (k = 1) – so the page isn’t empty on load           */
/* ------------------------------------------------------------------ */
drawGraph(1);
</script>
</body>
</html>
