# 🧮 HTML Calculator

A clean, modern, responsive calculator built with **plain HTML, CSS and JavaScript** — no libraries, no build step.

## ✨ Features

- Glassmorphism-style dark UI with a soft gradient background
- Addition, subtraction, multiplication, division and percentage
- Sign toggle (±), decimal point, clear (AC) and backspace (⌫)
- Shows the running expression above the result
- Full keyboard support (`0–9`, `+ - * /`, `.`, `%`, `Enter`/`=`, `Backspace`, `Esc`)
- Safe evaluation — no `eval()`
- Works on desktop and mobile

## 🚀 How to use

1. Copy the code below into a file named `index.html`
2. Open it in any web browser — that's it!

## 📄 Code

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Calculator</title>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@300;400;600&display=swap" rel="stylesheet" />
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }

    body {
      font-family: "Inter", system-ui, -apple-system, sans-serif;
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      background: radial-gradient(circle at 20% 20%, #3b2a7a 0%, transparent 45%),
                  radial-gradient(circle at 80% 80%, #0f5c6e 0%, transparent 45%),
                  #0d0f1a;
      padding: 20px;
    }

    .calculator {
      width: 100%;
      max-width: 360px;
      padding: 24px;
      border-radius: 28px;
      background: rgba(255, 255, 255, 0.06);
      border: 1px solid rgba(255, 255, 255, 0.12);
      backdrop-filter: blur(20px);
      -webkit-backdrop-filter: blur(20px);
      box-shadow: 0 30px 60px rgba(0, 0, 0, 0.45);
    }

    .display {
      text-align: right;
      padding: 16px 8px 24px;
      min-height: 120px;
      display: flex;
      flex-direction: column;
      justify-content: flex-end;
      overflow: hidden;
    }

    .expression {
      color: rgba(255, 255, 255, 0.45);
      font-size: 18px;
      min-height: 24px;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    .result {
      color: #fff;
      font-size: 52px;
      font-weight: 300;
      letter-spacing: -1px;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      transition: font-size 0.15s ease;
    }

    .keys {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 12px;
    }

    button {
      height: 68px;
      border: none;
      border-radius: 20px;
      font-family: inherit;
      font-size: 22px;
      font-weight: 400;
      color: #fff;
      cursor: pointer;
      background: rgba(255, 255, 255, 0.08);
      transition: background 0.15s ease, transform 0.08s ease;
      -webkit-tap-highlight-color: transparent;
    }

    button:hover  { background: rgba(255, 255, 255, 0.15); }
    button:active { transform: scale(0.94); }

    .fn       { color: #9fd3ff; background: rgba(159, 211, 255, 0.10); }
    .fn:hover { background: rgba(159, 211, 255, 0.18); }

    .op       { color: #fff; background: linear-gradient(135deg, #7c5cff, #5a8dff); font-weight: 600; }
    .op:hover { filter: brightness(1.12); background: linear-gradient(135deg, #7c5cff, #5a8dff); }
    .op.active { background: #fff; color: #5a6bff; }

    .equals       { background: linear-gradient(135deg, #00c6a7, #1e9bff); font-weight: 600; }
    .equals:hover { filter: brightness(1.1); background: linear-gradient(135deg, #00c6a7, #1e9bff); }

    .zero { grid-column: span 2; text-align: left; padding-left: 30px; }

    @media (max-width: 380px) {
      .calculator { padding: 18px; }
      button { height: 60px; font-size: 20px; }
      .result { font-size: 44px; }
    }
  </style>
</head>
<body>
  <main class="calculator">
    <div class="display">
      <div class="expression" id="expression"></div>
      <div class="result" id="result">0</div>
    </div>

    <div class="keys">
      <button class="fn" data-action="clear">AC</button>
      <button class="fn" data-action="sign">±</button>
      <button class="fn" data-action="percent">%</button>
      <button class="op" data-op="/">÷</button>

      <button data-num="7">7</button>
      <button data-num="8">8</button>
      <button data-num="9">9</button>
      <button class="op" data-op="*">×</button>

      <button data-num="4">4</button>
      <button data-num="5">5</button>
      <button data-num="6">6</button>
      <button class="op" data-op="-">−</button>

      <button data-num="1">1</button>
      <button data-num="2">2</button>
      <button data-num="3">3</button>
      <button class="op" data-op="+">+</button>

      <button class="zero" data-num="0">0</button>
      <button data-action="decimal">.</button>
      <button class="equals" data-action="equals">=</button>
    </div>
  </main>

  <script>
    const resultEl = document.getElementById("result");
    const exprEl = document.getElementById("expression");
    const symbols = { "+": "+", "-": "−", "*": "×", "/": "÷" };

    let current = "0";      // number being typed
    let previous = null;    // stored left operand
    let operator = null;    // pending operator
    let resetNext = false;  // start a new number on next digit

    function format(value) {
      if (value === "Error") return value;
      const n = Number(value);
      if (!isFinite(n)) return "Error";
      if (value.endsWith(".")) return Number(value.slice(0, -1)).toLocaleString("en-US") + ".";
      const [int, dec] = value.split(".");
      const intFmt = Number(int).toLocaleString("en-US");
      return dec !== undefined ? `${intFmt}.${dec}` : intFmt;
    }

    function render() {
      const text = format(current);
      resultEl.textContent = text;
      resultEl.style.fontSize = text.length > 12 ? "32px" : text.length > 9 ? "40px" : "";
      exprEl.textContent = operator ? `${format(previous)} ${symbols[operator]}` : exprEl.textContent;
      document.querySelectorAll(".op").forEach(b =>
        b.classList.toggle("active", b.dataset.op === operator && resetNext));
    }

    function compute(a, b, op) {
      a = parseFloat(a); b = parseFloat(b);
      let r;
      switch (op) {
        case "+": r = a + b; break;
        case "-": r = a - b; break;
        case "*": r = a * b; break;
        case "/": if (b === 0) return "Error"; r = a / b; break;
      }
      return String(parseFloat(r.toPrecision(12)));
    }

    function inputNumber(d) {
      if (current === "Error") clearAll();
      if (resetNext) { current = "0"; resetNext = false; }
      if (current.replace(/[-.]/g, "").length >= 15) return;
      current = current === "0" ? d : current === "-0" ? "-" + d : current + d;
    }

    function inputDecimal() {
      if (resetNext) { current = "0"; resetNext = false; }
      if (!current.includes(".")) current += ".";
    }

    function chooseOperator(op) {
      if (current === "Error") return;
      if (operator && !resetNext) {
        current = compute(previous, current, operator);
      }
      previous = current;
      operator = op;
      resetNext = true;
    }

    function equals() {
      if (!operator || current === "Error") return;
      const expr = `${format(previous)} ${symbols[operator]} ${format(current)} =`;
      current = compute(previous, current, operator);
      previous = null;
      operator = null;
      resetNext = true;
      exprEl.textContent = expr;
    }

    function clearAll() {
      current = "0"; previous = null; operator = null; resetNext = false;
      exprEl.textContent = "";
    }

    function backspace() {
      if (resetNext || current === "Error") return;
      current = current.length > 1 && current !== "-0" ? current.slice(0, -1) : "0";
      if (current === "-") current = "0";
    }

    function toggleSign() {
      if (current === "Error") return;
      current = current.startsWith("-") ? current.slice(1) : "-" + current;
    }

    function percent() {
      if (current === "Error") return;
      current = String(parseFloat((parseFloat(current) / 100).toPrecision(12)));
    }

    document.querySelector(".keys").addEventListener("click", e => {
      const btn = e.target.closest("button");
      if (!btn) return;
      const { num, op, action } = btn.dataset;
      if (num !== undefined) inputNumber(num);
      else if (op) chooseOperator(op);
      else if (action === "decimal") inputDecimal();
      else if (action === "equals") equals();
      else if (action === "clear") clearAll();
      else if (action === "sign") toggleSign();
      else if (action === "percent") percent();
      render();
    });

    document.addEventListener("keydown", e => {
      const k = e.key;
      if (/^[0-9]$/.test(k)) inputNumber(k);
      else if (["+", "-", "*", "/"].includes(k)) chooseOperator(k);
      else if (k === ".") inputDecimal();
      else if (k === "Enter" || k === "=") { e.preventDefault(); equals(); }
      else if (k === "Backspace") backspace();
      else if (k === "Escape") clearAll();
      else if (k === "%") percent();
      else return;
      render();
    });

    render();
  </script>
</body>
</html>
```

## 🛠️ Built with

- HTML5
- CSS3 (Grid, gradients, backdrop-filter)
- Vanilla JavaScript
