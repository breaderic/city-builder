<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Kurima Dashboard</title>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/10.12.2/firebase-database-compat.js"></script>
</head>
<body style="margin:0; background:#0d0c14;">
<div id="village-root"></div>
<style>
  #village-root {
    --ink: #171225;
    --ink-soft: #221A33;
    --vellum: #E8D6A8;
    --vellum-soft: #DEC788;
    --vellum-line: #C2A768;
    --text-dark: #3A2A12;
    --text-dim: #6B5730;
    --labor: #B1481F;
    --labor-deep: #7E3115;
    --favor: #1F7A6C;
    --favor-deep: #145349;
    --trade: #C79A3E;
    --unmet: #7A2A2A;
    --arcane: #5B4B8A;
    --arcane-deep: #3E3560;

    font-family: 'Spectral', Georgia, serif;
    color: var(--text-dark);
    background: var(--ink);
    background-image:
      radial-gradient(ellipse at 15% 0%, rgba(199,154,62,0.16), transparent 55%),
      radial-gradient(ellipse at 88% 10%, rgba(31,122,108,0.16), transparent 50%);
    padding: 28px;
    border-radius: 10px;
    max-width: 1100px;
    margin: 0 auto;
    box-sizing: border-box;
  }
  #village-root * { box-sizing: border-box; }

  .vb-header {
    display: flex;
    align-items: baseline;
    justify-content: space-between;
    gap: 16px;
    margin-bottom: 18px;
    flex-wrap: wrap;
  }
  .vb-title {
    font-family: 'Fraunces', Georgia, serif;
    font-weight: 600;
    font-size: 34px;
    color: var(--vellum);
    letter-spacing: 0.2px;
    background: transparent;
    border: none;
    outline: none;
    padding: 0;
    max-width: 500px;
  }
  .vb-title::placeholder { color: rgba(232,214,168,0.45); }
  .vb-subtitle {
    font-size: 14px;
    color: rgba(232,214,168,0.6);
    font-style: italic;
    max-width: 340px;
    text-align: right;
  }
  .vb-gm-toggle {
    font-family: 'Spectral', serif;
    font-size: 11.5px;
    padding: 5px 10px;
    border-radius: 4px;
    border: 1px solid rgba(232,214,168,0.35);
    background: transparent;
    color: rgba(232,214,168,0.7);
    cursor: pointer;
    white-space: nowrap;
  }
  .vb-gm-toggle:hover { background: rgba(232,214,168,0.1); }
  .vb-gm-toggle.active {
    background: rgba(232,214,168,0.18);
    color: var(--vellum);
    border-color: var(--vellum-line);
  }

  /* ---- Hero skyline: procedural placeholder, structured so a real map/art
     image can later replace or layer beneath #vb-skyline-art ---- */
  .vb-hero {
    position: relative;
    height: 180px;
    border-radius: 8px;
    overflow: hidden;
    margin-bottom: 26px;
    background: linear-gradient(180deg, #221933 0%, #17111f 70%, #100c17 100%);
    border: 1px solid rgba(232,214,168,0.12);
  }
  #vb-skyline-art { position:absolute; inset:0; }
  .vb-hero-caption {
    position: absolute;
    bottom: 10px;
    left: 14px;
    font-size: 12px;
    color: rgba(232,214,168,0.5);
    font-family: 'Spectral', serif;
    font-style: italic;
  }

  /* ---- Layout ---- */
  .vb-grid {
    display: grid;
    grid-template-columns: 260px 1fr;
    gap: 22px;
  }
  @media (max-width: 720px) {
    .vb-grid { grid-template-columns: 1fr; }
  }

  .vb-panel {
    background: var(--vellum);
    border-radius: 6px;
    padding: 18px;
    border: 1px solid var(--vellum-line);
  }
  .vb-panel + .vb-panel { margin-top: 18px; }
  .vb-panel h3 {
    font-family: 'Fraunces', Georgia, serif;
    font-size: 15px;
    font-weight: 600;
    margin: 0 0 14px 0;
    color: var(--text-dark);
    border-bottom: 1px solid var(--vellum-line);
    padding-bottom: 8px;
  }

  /* ---- Resource gauges ---- */
  .vb-resource { margin-bottom: 16px; }
  .vb-resource:last-child { margin-bottom: 0; }
  .vb-resource-row {
    display: flex;
    justify-content: space-between;
    align-items: baseline;
    font-size: 13px;
    margin-bottom: 5px;
  }
  .vb-resource-name { font-weight: 600; }
  .vb-resource-val { font-variant-numeric: tabular-nums; color: var(--text-dim); min-width: 20px; text-align: center; display: inline-block; }
  .vb-resource-controls { display: flex; align-items: center; gap: 6px; }
  .vb-step-btn {
    width: 18px;
    height: 18px;
    line-height: 16px;
    padding: 0;
    font-size: 13px;
    font-family: 'Spectral', serif;
    border-radius: 50%;
    border: 1px solid var(--vellum-line);
    background: var(--vellum-soft);
    color: var(--text-dark);
    cursor: pointer;
  }
  .vb-step-btn:hover { background: var(--vellum-line); }
  .vb-step-btn:active { transform: translateY(1px); }
  .vb-gauge-track {
    height: 10px;
    background: rgba(58,42,18,0.14);
    border-radius: 5px;
    overflow: hidden;
    position: relative;
  }
  .vb-gauge-fill {
    height: 100%;
    border-radius: 5px;
    transition: width 0.6s cubic-bezier(.4,0,.2,1);
  }
  .vb-gauge-fill.labor { background: var(--labor); }
  .vb-gauge-fill.trade { background: var(--trade); }
  .vb-gauge-fill.favor { background: var(--favor); }
  .vb-resource-income { font-size: 11px; color: var(--text-dim); margin-top: 3px; }

  .vb-add-row { display:flex; gap:6px; margin-top: 10px; }
  .vb-add-row button {
    flex: 1;
    font-family: 'Spectral', serif;
    font-size: 12px;
    padding: 5px 0;
    background: var(--vellum-soft);
    border: 1px solid var(--vellum-line);
    border-radius: 4px;
    cursor: pointer;
    color: var(--text-dark);
  }
  .vb-add-row button:hover { background: var(--vellum-line); }
  .vb-add-row button:active { transform: translateY(1px); }

  /* ---- Building board ---- */
  .vb-tier { margin-bottom: 22px; }
  .vb-tier:last-child { margin-bottom: 0; }
  .vb-tier-label {
    font-family: 'Fraunces', Georgia, serif;
    font-size: 13px;
    color: var(--text-dim);
    margin-bottom: 10px;
    display: flex;
    align-items: center;
    gap: 8px;
  }
  .vb-tier-line {
    flex: 1;
    height: 1px;
    background: var(--vellum-line);
  }
  .vb-tier-teaser {
    font-size: 12.5px;
    color: var(--text-dim);
    font-style: italic;
    padding: 4px 0 2px 0;
  }
  .vb-cards {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
    gap: 12px;
  }
  .vb-card {
    background: var(--vellum-soft);
    border: 1px solid var(--vellum-line);
    border-radius: 6px;
    padding: 12px;
    position: relative;
    transition: box-shadow 0.2s, transform 0.2s;
  }
  .vb-card.locked { opacity: 0.5; }
  .vb-card.built {
    background: linear-gradient(160deg, #F1E4BC, var(--vellum-soft));
    border-color: var(--labor);
  }
  .vb-card.affordable:hover { box-shadow: 0 3px 10px rgba(58,42,18,0.18); }
  .vb-card-name {
    font-family: 'Fraunces', Georgia, serif;
    font-weight: 600;
    font-size: 14px;
    margin: 0 0 4px 0;
  }
  .vb-card-name[contenteditable="true"] { outline: none; cursor: text; }
  .vb-card-status {
    font-size: 10.5px;
    color: var(--text-dim);
    margin-bottom: 8px;
  }
  .vb-card-cost {
    display: flex;
    flex-wrap: wrap;
    gap: 6px;
    font-size: 11.5px;
    color: var(--text-dim);
    margin-bottom: 6px;
  }
  .vb-card-cost span.met { color: var(--labor-deep); font-weight: 600; }
  .vb-card-cost span.unmet { color: var(--unmet); }
  .vb-card-yield {
    font-size: 11px;
    color: var(--text-dim);
    margin-bottom: 6px;
    font-style: italic;
  }
  .vb-card-boon {
    font-size: 11.5px;
    color: var(--favor-deep);
    background: rgba(31,122,108,0.12);
    border: 1px solid rgba(31,122,108,0.3);
    border-radius: 4px;
    padding: 6px 8px;
    margin-bottom: 10px;
    line-height: 1.4;
  }
  .vb-card-btn {
    width: 100%;
    padding: 6px 0;
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    border-radius: 4px;
    border: 1px solid var(--labor-deep);
    background: var(--labor);
    color: var(--vellum);
    cursor: pointer;
  }
  .vb-card-btn:disabled {
    background: transparent;
    color: var(--text-dim);
    border-color: var(--vellum-line);
    cursor: not-allowed;
  }
  .vb-card-btn:not(:disabled):hover { background: var(--labor-deep); }
  .vb-card.just-built { animation: vb-reveal 0.7s ease; }
  @keyframes vb-reveal {
    0% { transform: scale(0.94); box-shadow: 0 0 0 rgba(199,154,62,0); }
    45% { transform: scale(1.03); box-shadow: 0 0 22px rgba(199,154,62,0.5); }
    100% { transform: scale(1); box-shadow: 0 0 0 rgba(199,154,62,0); }
  }

  /* ---- Rumor / NPC log ---- */
  .vb-log { max-height: 260px; overflow-y: auto; padding-right: 4px; }
  .vb-log-entry {
    padding: 10px 0;
    border-bottom: 1px solid var(--vellum-line);
    font-size: 13px;
    line-height: 1.5;
  }
  .vb-log-entry:last-child { border-bottom: none; }
  .vb-log-entry .vb-log-source {
    display: block;
    font-family: 'Fraunces', Georgia, serif;
    font-weight: 600;
    font-size: 12.5px;
    color: var(--labor-deep);
    margin-bottom: 2px;
  }
  .vb-log-empty { color: var(--text-dim); font-style: italic; font-size: 13px; }

  /* ---- Party chat (shared, persistent) ---- */
  .vb-chat-note {
    font-size: 11px;
    color: var(--text-dim);
    font-style: italic;
    margin-bottom: 10px;
  }
  .vb-chat-log {
    max-height: 200px;
    overflow-y: auto;
    margin-bottom: 10px;
    padding-right: 4px;
  }
  .vb-chat-msg {
    font-size: 12.5px;
    line-height: 1.5;
    padding: 5px 0;
    border-bottom: 1px solid var(--vellum-line);
  }
  .vb-chat-msg:last-child { border-bottom: none; }
  .vb-chat-name {
    font-family: 'Fraunces', Georgia, serif;
    font-weight: 600;
    color: var(--labor-deep);
    margin-right: 6px;
  }
  .vb-chat-text { color: var(--text-dark); }
  .vb-chat-form input {
    width: 100%;
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    color: var(--text-dark);
    background: rgba(255,255,255,0.35);
    border: 1px solid var(--vellum-line);
    border-radius: 4px;
    padding: 6px 8px;
    margin-bottom: 6px;
  }
  .vb-chat-input-row { display: flex; gap: 6px; }
  .vb-chat-input-row input { margin-bottom: 0; flex: 1; }
  .vb-chat-input-row button {
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    padding: 6px 14px;
    border-radius: 4px;
    border: 1px solid var(--favor-deep);
    background: var(--favor);
    color: var(--vellum);
    cursor: pointer;
  }
  .vb-chat-input-row button:hover { background: var(--favor-deep); }

  /* ---- Notes & Images (shared, persistent, organized by date) ---- */
  .vb-notes-note {
    font-size: 11px;
    color: rgba(232,214,168,0.55);
    font-style: italic;
    margin-bottom: 12px;
  }
  .vb-notes-composer {
    background: rgba(232,214,168,0.06);
    border: 1px solid rgba(232,214,168,0.2);
    border-radius: 6px;
    padding: 12px;
    margin-bottom: 16px;
  }
  .vb-notes-composer textarea {
    width: 100%;
    min-height: 50px;
    resize: vertical;
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    color: var(--vellum);
    background: rgba(232,214,168,0.08);
    border: 1px solid rgba(232,214,168,0.25);
    border-radius: 4px;
    padding: 8px;
    margin-bottom: 8px;
  }
  .vb-notes-composer textarea::placeholder { color: rgba(232,214,168,0.4); }
  .vb-notes-composer-row {
    display: flex;
    gap: 8px;
    align-items: center;
    margin-bottom: 8px;
    flex-wrap: wrap;
  }
  .vb-notes-composer-row:last-child { margin-bottom: 0; }
  .vb-notes-composer-row input[type="text"],
  .vb-notes-composer-row input:not([type="file"]) {
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    color: var(--vellum);
    background: rgba(232,214,168,0.08);
    border: 1px solid rgba(232,214,168,0.25);
    border-radius: 4px;
    padding: 6px 8px;
    flex: 1;
    min-width: 120px;
  }
  .vb-notes-composer-row input[type="file"] {
    font-size: 11.5px;
    color: rgba(232,214,168,0.7);
    flex: 1;
    min-width: 160px;
  }
  .vb-notes-composer-row button {
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    padding: 6px 14px;
    border-radius: 4px;
    border: 1px solid var(--favor-deep);
    background: var(--favor);
    color: var(--vellum);
    cursor: pointer;
    white-space: nowrap;
  }
  .vb-notes-composer-row button:hover { background: var(--favor-deep); }
  .vb-notes-body {
    display: grid;
    grid-template-columns: 150px 1fr;
    gap: 16px;
  }
  @media (max-width: 600px) {
    .vb-notes-body { grid-template-columns: 1fr; }
  }
  .vb-notes-sidebar {
    border-right: 1px solid rgba(232,214,168,0.18);
    padding-right: 12px;
    max-height: 360px;
    overflow-y: auto;
  }
  .vb-notes-date-item {
    font-size: 12px;
    color: rgba(232,214,168,0.65);
    padding: 6px 8px;
    border-radius: 4px;
    cursor: pointer;
    display: flex;
    justify-content: space-between;
    gap: 6px;
    margin-bottom: 2px;
  }
  .vb-notes-date-item:hover { background: rgba(232,214,168,0.08); }
  .vb-notes-date-item.active {
    background: rgba(232,214,168,0.16);
    color: var(--vellum);
    font-weight: 600;
  }
  .vb-notes-count { color: rgba(232,214,168,0.4); font-variant-numeric: tabular-nums; }
  .vb-notes-main {
    max-height: 360px;
    overflow-y: auto;
    padding-right: 4px;
  }
  .vb-note-card {
    position: relative;
    background: rgba(232,214,168,0.06);
    border: 1px solid rgba(232,214,168,0.16);
    border-radius: 6px;
    padding: 10px 30px 10px 12px;
    margin-bottom: 10px;
  }
  .vb-note-card:last-child { margin-bottom: 0; }
  .vb-note-text {
    font-size: 13px;
    color: var(--vellum);
    line-height: 1.5;
    white-space: pre-wrap;
    margin-bottom: 6px;
  }
  .vb-note-image {
    max-width: 100%;
    max-height: 220px;
    border-radius: 4px;
    display: block;
    margin-bottom: 6px;
  }
  .vb-note-caption {
    font-size: 12px;
    font-style: italic;
    color: rgba(232,214,168,0.75);
    margin-bottom: 6px;
  }
  .vb-note-meta {
    font-size: 10.5px;
    color: rgba(232,214,168,0.45);
  }
  .vb-note-delete {
    position: absolute;
    top: 8px;
    right: 8px;
    background: none;
    border: none;
    color: rgba(232,214,168,0.35);
    cursor: pointer;
    font-size: 12px;
  }
  .vb-note-delete:hover { color: var(--unmet); }

  /* ---- Character sheets ---- */
  .vb-char-item {
    font-size: 12.5px;
    color: var(--text-dark);
    padding: 8px 8px;
    border-radius: 4px;
    cursor: pointer;
    margin-bottom: 3px;
  }
  .vb-char-item:hover { background: var(--vellum-line); }
  .vb-char-item.active { background: var(--vellum-line); font-weight: 600; }
  .vb-char-item-name { font-family: 'Fraunces', Georgia, serif; font-weight: 600; }
  .vb-char-item-owner { font-size: 10.5px; color: var(--text-dim); }

  .vb-char-form { font-size: 12.5px; }
  .vb-cf-row {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(110px, 1fr));
    gap: 10px;
    margin-bottom: 12px;
  }
  .vb-cf-label {
    display: flex;
    flex-direction: column;
    gap: 4px;
    font-size: 11px;
    color: var(--text-dim);
    font-weight: 600;
  }
  .vb-cf-label-wide { grid-column: 1 / -1; }
  .vb-cf-label input, .vb-cf-label textarea {
    font-family: 'Spectral', serif;
    font-size: 13px;
    font-weight: 400;
    color: var(--text-dark);
    background: rgba(255,255,255,0.4);
    border: 1px solid var(--vellum-line);
    border-radius: 4px;
    padding: 6px 8px;
  }
  .vb-cf-label textarea { min-height: 60px; resize: vertical; }
  .vb-cf-view {
    display: flex;
    flex-direction: column;
    gap: 2px;
  }
  .vb-cf-view-wide { grid-column: 1 / -1; }
  .vb-cf-view-label { font-size: 11px; color: var(--text-dim); font-weight: 600; }
  .vb-cf-view-value { font-size: 13.5px; }
  .vb-cf-view-text { font-size: 13px; white-space: pre-wrap; line-height: 1.5; }

  .vb-cf-tracks {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
    gap: 14px;
    margin-bottom: 14px;
  }
  .vb-track-label {
    font-size: 11px;
    color: var(--text-dim);
    font-weight: 600;
    margin-bottom: 4px;
    display: flex;
    align-items: center;
    gap: 6px;
  }
  .vb-track-max {
    width: 44px;
    font-family: 'Spectral', serif;
    font-size: 11px;
    border: 1px solid var(--vellum-line);
    border-radius: 3px;
    padding: 1px 4px;
  }
  .vb-pip-track { display: flex; flex-wrap: wrap; gap: 4px; }
  .vb-pip {
    width: 14px;
    height: 14px;
    border-radius: 3px;
    border: 1px solid var(--vellum-line);
    background: rgba(255,255,255,0.4);
    display: inline-block;
  }
  .vb-pip.filled { background: var(--labor); border-color: var(--labor-deep); }
  .vb-pip.clickable { cursor: pointer; }
  .vb-pip.clickable:hover { border-color: var(--labor-deep); }

  .vb-char-view-header {
    font-family: 'Fraunces', Georgia, serif;
    font-size: 16px;
    font-weight: 600;
    margin-bottom: 12px;
    display: flex;
    align-items: center;
    gap: 10px;
  }
  .vb-char-view-owner { font-size: 11.5px; color: var(--text-dim); font-weight: 400; font-style: italic; }
  .vb-char-unlock-row { display: flex; gap: 8px; margin-top: 10px; }
  .vb-char-unlock-row input {
    flex: 1;
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    border: 1px solid var(--vellum-line);
    border-radius: 4px;
    padding: 6px 8px;
    background: rgba(255,255,255,0.4);
  }
  .vb-char-unlock-row button, .vb-char-save-btn, .vb-char-lock-btn {
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    padding: 6px 14px;
    border-radius: 4px;
    border: 1px solid var(--labor-deep);
    background: var(--labor);
    color: var(--vellum);
    cursor: pointer;
  }
  .vb-char-unlock-row button:hover, .vb-char-save-btn:hover { background: var(--labor-deep); }
  .vb-char-lock-btn {
    font-size: 11px;
    padding: 3px 10px;
    background: transparent;
    color: var(--text-dim);
    border-color: var(--vellum-line);
  }
  .vb-char-lock-btn:hover { background: var(--vellum-line); }
  .vb-char-save-btn { margin-top: 4px; }

  .vb-char-body {
    display: grid;
    grid-template-columns: 160px 1fr;
    gap: 16px;
  }
  @media (max-width: 600px) {
    .vb-char-body { grid-template-columns: 1fr; }
  }
  .vb-char-sidebar {
    border-right: 1px solid var(--vellum-line);
    padding-right: 12px;
    max-height: 480px;
    overflow-y: auto;
  }
  .vb-char-main {
    max-height: 480px;
    overflow-y: auto;
    padding-right: 4px;
  }
  .vb-char-password-note {
    font-size: 11px;
    color: var(--text-dim);
    font-style: italic;
    margin-bottom: 12px;
  }

  /* ---- Domain card picker (nested in the character sheet) ---- */
  .vb-domain-picker {
    grid-column: 1 / -1;
    background: rgba(91,75,138,0.05);
    border: 1px solid rgba(91,75,138,0.25);
    border-radius: 6px;
    padding: 4px 12px;
    margin-bottom: 12px;
  }
  .vb-domain-picker summary {
    cursor: pointer;
    font-family: 'Fraunces', Georgia, serif;
    font-size: 12.5px;
    font-weight: 600;
    color: var(--arcane-deep);
    padding: 8px 0;
  }
  .vb-domain-block-nested { padding-bottom: 10px; }
  .vb-domain-title-nested {
    font-size: 11.5px;
    font-weight: 600;
    color: var(--text-dim);
    margin-bottom: 8px;
  }
  .vb-domain-level-group { margin-bottom: 10px; }
  .vb-domain-level-label {
    font-size: 11px;
    font-weight: 600;
    color: var(--text-dim);
    margin-bottom: 5px;
  }
  .vb-domain-card-row { display: flex; flex-wrap: wrap; gap: 6px; }
  .vb-domain-card-chip {
    font-size: 12px;
    padding: 4px 10px;
    border-radius: 999px;
    background: rgba(91,75,138,0.1);
    border: 1px solid rgba(91,75,138,0.35);
    color: var(--arcane-deep);
  }
  .vb-domain-card-chip.clickable { cursor: pointer; }
  .vb-domain-card-chip.clickable:hover { background: rgba(91,75,138,0.22); }

  /* ---- Post-a-job form ---- */
  .vb-post-form { margin-bottom: 14px; }
  .vb-post-form textarea {
    width: 100%;
    min-height: 56px;
    resize: vertical;
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    color: var(--text-dark);
    background: rgba(255,255,255,0.35);
    border: 1px solid var(--vellum-line);
    border-radius: 4px;
    padding: 8px;
    margin-bottom: 6px;
  }
  .vb-post-form textarea::placeholder { color: var(--text-dim); }
  .vb-post-btn {
    width: 100%;
    padding: 6px 0;
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    border-radius: 4px;
    border: 1px solid var(--favor-deep);
    background: var(--favor);
    color: var(--vellum);
    cursor: pointer;
  }
  .vb-post-btn:hover { background: var(--favor-deep); }

  /* ---- Quest parchment card ---- */
  .vb-quest-card {
    position: relative;
    background:
      radial-gradient(circle at 85% 15%, rgba(122,42,42,0.05), transparent 40%),
      linear-gradient(165deg, #F3E6BE, #E7D6A2);
    border: 1px solid #B99A5C;
    border-radius: 3px 9px 4px 10px / 7px 3px 11px 4px;
    box-shadow: 0 2px 6px rgba(23,18,37,0.25);
    padding: 14px 16px 12px 16px;
    margin-bottom: 14px;
    transform: rotate(-0.4deg);
    font-family: 'Spectral', serif;
  }
  .vb-quest-card:nth-child(even) { transform: rotate(0.5deg); }
  .vb-quest-card::before {
    content: '';
    position: absolute;
    top: -6px;
    right: 16px;
    width: 18px;
    height: 18px;
    border-radius: 50%;
    background: radial-gradient(circle at 35% 30%, #9B3B2E, #6E2119 70%);
    box-shadow: 0 1px 2px rgba(0,0,0,0.4);
  }
  .vb-quest-label {
    font-size: 10.5px;
    letter-spacing: 0.3px;
    color: var(--text-dim);
    font-style: italic;
    margin-bottom: 6px;
  }
  .vb-quest-text {
    font-size: 13.5px;
    line-height: 1.55;
    color: var(--text-dark);
    white-space: pre-wrap;
  }
  .vb-quest-remove {
    position: absolute;
    bottom: 8px;
    right: 10px;
    font-size: 11px;
    color: var(--text-dim);
    background: none;
    border: none;
    cursor: pointer;
    text-decoration: underline;
    padding: 2px;
  }
  .vb-quest-remove:hover { color: var(--unmet); }

  /* ---- GM-only review panel (hidden from players by default) ---- */
  .vb-gm-panel {
    margin-top: 20px;
    background: var(--ink-soft);
    border: 1px dashed rgba(232,214,168,0.35);
    border-radius: 8px;
    padding: 16px 18px;
  }
  .vb-gm-panel-label {
    font-family: 'Fraunces', Georgia, serif;
    font-size: 13px;
    font-weight: 600;
    color: var(--vellum);
    margin-bottom: 3px;
  }
  .vb-gm-panel-sub {
    font-size: 11.5px;
    color: rgba(232,214,168,0.55);
    margin-bottom: 14px;
    font-style: italic;
  }
  .vb-gm-row {
    display: grid;
    grid-template-columns: 130px 1fr 32px;
    gap: 8px;
    align-items: start;
    padding: 8px 0;
    border-bottom: 1px solid rgba(232,214,168,0.14);
  }
  .vb-gm-row:last-of-type { border-bottom: none; }
  .vb-gm-row input, .vb-gm-row textarea {
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    color: var(--vellum);
    background: rgba(232,214,168,0.08);
    border: 1px solid rgba(232,214,168,0.25);
    border-radius: 4px;
    padding: 6px 8px;
    resize: vertical;
    min-height: 34px;
  }
  .vb-gm-row input::placeholder, .vb-gm-row textarea::placeholder { color: rgba(232,214,168,0.35); }
  .vb-gm-tag {
    font-size: 10px;
    text-transform: lowercase;
    color: rgba(232,214,168,0.45);
    align-self: center;
  }
  .vb-gm-delete {
    font-size: 15px;
    background: none;
    border: none;
    color: rgba(232,214,168,0.5);
    cursor: pointer;
    align-self: center;
  }
  .vb-gm-delete:hover { color: var(--unmet); }
  .vb-gm-empty { color: rgba(232,214,168,0.4); font-style: italic; font-size: 12.5px; padding: 4px 0; }
  .vb-gm-add {
    margin-top: 14px;
    padding-top: 14px;
    border-top: 1px solid rgba(232,214,168,0.2);
    display: flex;
    flex-direction: column;
    gap: 8px;
  }
  .vb-gm-add input, .vb-gm-add select, .vb-gm-add textarea {
    font-family: 'Spectral', serif;
    font-size: 12.5px;
    color: var(--vellum);
    background: rgba(232,214,168,0.08);
    border: 1px solid rgba(232,214,168,0.25);
    border-radius: 4px;
    padding: 6px 8px;
  }
  .vb-gm-add textarea { min-height: 60px; resize: vertical; }
  .vb-gm-add input::placeholder, .vb-gm-add textarea::placeholder { color: rgba(232,214,168,0.35); }
  .vb-gm-add select { cursor: pointer; }
  .vb-gm-add-btn, .vb-gm-save-btn {
    font-family: 'Spectral', serif;
    font-size: 12px;
    padding: 6px 12px;
    border-radius: 4px;
    background: transparent;
    color: var(--vellum);
    cursor: pointer;
    align-self: flex-start;
  }
  .vb-gm-add-btn { border: 1px solid var(--favor); }
  .vb-gm-add-btn:hover { background: rgba(31,122,108,0.25); }
  .vb-gm-save-btn { border: 1px solid var(--trade); }
  .vb-gm-save-btn:hover { background: rgba(199,154,62,0.22); }

  .vb-footnote {
    margin-top: 16px;
    font-size: 11.5px;
    color: rgba(232,214,168,0.45);
    text-align: center;
  }
</style>

<script>
(function () {
  const firebaseConfig = {
    apiKey: "AIzaSyDy-oq_FY5XYFwMDiWukkkFX4eo8lYvOX4",
    authDomain: "citybuilder-kurima.firebaseapp.com",
    databaseURL: "https://citybuilder-kurima-default-rtdb.firebaseio.com",
    projectId: "citybuilder-kurima",
    storageBucket: "citybuilder-kurima.firebasestorage.app",
    messagingSenderId: "1068607737045",
    appId: "1:1068607737045:web:3c4f6fb41c5c0053a3a65c"
  };
  firebase.initializeApp(firebaseConfig);
  const fbDb = firebase.database();

  // Drop-in replacement for the old Claude-artifact window.storage API.
  // shared=true  -> synced live for everyone via Firebase Realtime Database
  // shared=false -> stays local to this browser via localStorage
  const LOCAL_PREFIX = 'kurima-dashboard:';
  const storage = {
    get: function (key, shared) {
      if (shared) {
        return fbDb.ref('kurima-dashboard/' + key).once('value').then(function (snap) {
          const v = snap.val();
          if (v === null || v === undefined) throw new Error('not found');
          return { key: key, value: v, shared: true };
        });
      }
      const v = localStorage.getItem(LOCAL_PREFIX + key);
      if (v === null) return Promise.reject(new Error('not found'));
      return Promise.resolve({ key: key, value: v, shared: false });
    },
    set: function (key, value, shared) {
      if (shared) {
        return fbDb.ref('kurima-dashboard/' + key).set(value).then(function () {
          return { key: key, value: value, shared: true };
        });
      }
      localStorage.setItem(LOCAL_PREFIX + key, value);
      return Promise.resolve({ key: key, value: value, shared: false });
    },
    delete: function (key, shared) {
      if (shared) {
        return fbDb.ref('kurima-dashboard/' + key).remove().then(function () {
          return { key: key, deleted: true, shared: true };
        });
      }
      localStorage.removeItem(LOCAL_PREFIX + key);
      return Promise.resolve({ key: key, deleted: true, shared: false });
    }
  };

  const root = document.getElementById('village-root');

  const RESOURCE_META = {
    labor:  { label: 'Labor',  cls: 'labor' },
    trade:  { label: 'Trade',  cls: 'trade'  },
    favor:  { label: 'Favor',  cls: 'favor' },
  };
  const GAUGE_CAP = 100;

  // Domain reference data, built from uploaded Daggerheart domain/ability text.
  // Add more entries here as more domains get uploaded.
  const DOMAIN_LIBRARY = {
    arcana: {
      name: 'Arcana',
      classes: ['Druid', 'Sorcerer'],
      description: 'Arcana is the domain of innate and instinctual magic. Those who choose this path tap into the raw, enigmatic forces of the realms to manipulate both their own energy and the elements. Arcana offers wielders a volatile power, but it is incredibly potent when correctly channeled.',
      cards: [
        { level: 1, options: ['Rune Ward', 'Unleash Chaos', 'Wall Walk'] },
        { level: 2, options: ['Cinder Grasp', 'Floating Eye'] },
        { level: 3, options: ['Counterspell', 'Flight'] },
        { level: 4, options: ['Blink Out', 'Preservation Blast'] },
        { level: 5, options: ['Chain Lightning', 'Premonition'] },
        { level: 6, options: ['Rift Walker', 'Telekinesis'] },
        { level: 7, options: ['Arcana-Touched', 'Cloaking Blast'] },
        { level: 8, options: ['Arcane Reflection', 'Confusing Aura'] },
        { level: 9, options: ['Earthquake', 'Sensory Projection'] },
        { level: 10, options: ['Adjust Reality', 'Falling Sky'] },
      ],
    },
  };

  const DEFAULT_STATE = {
    villageName: 'Kurima',
    tagline: 'an oasis, and everything it could become',
    resources: { labor: 8, trade: 14, favor: 5 },
    income: { labor: 0, trade: 0, favor: 0 },
    buildings: [
      // --- Tier 1: Oasis Camp ---
      { id: 'well-first-light', tier: 1, name: 'Well of First Light', npcName: 'Rasha the Diviner', cost: { trade: 8 }, yields: { labor: 2 },
        hook: "An old diviner named Rasha claims the well answers to her rod alone, and won't say why.", state: 'available' },
      { id: 'palm-terraces', tier: 1, name: 'Palm Grove Terraces', npcName: 'The Grove-Keeper Family', cost: { trade: 10 }, yields: { trade: 2 },
        hook: 'A grove-keeper family moves in overnight, insisting the palms were planted by their ancestors.', state: 'available' },
      { id: 'sun-shrine', tier: 1, name: 'Sun-Shrine of Three Roads', npcName: 'The Blind Pilgrim', cost: { trade: 6, favor: 4 }, yields: { favor: 1 },
        hook: 'A blind pilgrim settles at the shrine, reciting warnings no one asked to hear.', state: 'available' },
      { id: 'caravan-stakes', tier: 1, name: 'Caravanserai Stakes', npcName: 'The Camel-Driver Innkeeper', cost: { trade: 8 }, yields: { trade: 2 },
        hook: "A retired camel-driver stakes a claim to be Kurima's first innkeeper.", state: 'available' },
      { id: 'mudbrick-kiln', tier: 1, name: 'Mudbrick Kiln', npcName: 'The Kiln-Master', cost: { labor: 6, trade: 4 }, yields: { labor: 2 },
        hook: 'A kiln-master arrives bearing a rival\'s burn scars and a grudge she won\'t explain.', state: 'available' },
      { id: 'salt-camp', tier: 1, name: 'Salt Flats Camp', npcName: 'The Rival Salt-Traders', cost: { trade: 10 }, yields: { trade: 2 },
        boon: 'Once discovered, the party can trade at the salt flats even during sandstorms that close every other route.',
        hook: "Salt-traders from a rival oasis arrive first, and act like they already own the flats.", state: 'available' },
      { id: 'tannery-pits', tier: 1, name: 'Tannery Pits', npcName: 'The Leatherworker', cost: { labor: 8 }, yields: { labor: 1, trade: 1 },
        hook: 'The stench draws complaints &mdash; and a leatherworker offering to fix it, for favors owed later.', state: 'available' },
      { id: 'star-readers-tent', tier: 1, name: "Star-Reader's Tent", npcName: 'The Star-Reader', cost: { trade: 6, favor: 6 }, yields: { favor: 1 },
        boon: 'Once per session, the party may ask the star-reader for the safest route across a stretch of open desert.',
        hook: "A star-reader arrives uninvited, having already mapped Kurima's future in the sand.", state: 'available' },
      { id: 'watering-circle', tier: 1, name: 'Watering Trough Circle', npcName: 'The Herder', cost: { labor: 5, trade: 5 }, yields: { labor: 1 },
        boon: 'Mounts and pack animals stabled within Kurima recover fully overnight, storm or no.',
        hook: 'A herder claims the circle as neutral ground, and expects Kurima to honor it.', state: 'available' },

      // --- Tier 2: Trade Town ---
      { id: 'grand-cistern', tier: 2, name: 'The Grand Cistern', npcName: 'The Exiled Engineer', cost: { labor: 15, trade: 12 }, yields: { trade: 3 },
        boon: 'Kurima no longer suffers thirst penalties during the dry season, for townsfolk or travelers passing through.',
        hook: 'An engineer from a fallen city offers to build it &mdash; if Kurima will overlook her past.', state: 'locked' },
      { id: 'caravanserai-proper', tier: 2, name: 'Caravanserai Proper', npcName: 'The Caravan-Master', cost: { labor: 10, trade: 15 }, yields: { trade: 3 },
        hook: 'A caravan-master relocates her entire operation to Kurima, sight unseen.', state: 'locked' },
      { id: 'glassblowers-row', tier: 2, name: "Glassblowers' Row", npcName: 'The Glassblower', cost: { trade: 18 }, yields: { trade: 3 },
        hook: "A glassblower arrives claiming sand from Kurima's dunes makes glass that hums.", state: 'locked' },
      { id: 'falconers-mews', tier: 2, name: "Falconers' Mews", npcName: 'The Falconer', cost: { trade: 10, favor: 8 }, yields: { favor: 2 },
        boon: "The party may send a trained messenger falcon to any settlement they've visited, arriving within three days.",
        hook: "A falconer arrives with a bird that won't fly for anyone but her.", state: 'locked' },
      { id: 'sandstone-barracks', tier: 2, name: 'Sandstone Barracks', npcName: 'The Disgraced Officer', cost: { labor: 18, favor: 6 }, yields: { labor: 2, favor: 1 },
        boon: 'Kurima gains a standing militia, sharply cutting the odds that a raid succeeds unnoticed.',
        hook: 'A disgraced officer arrives asking to command the barracks &mdash; no questions, please.', state: 'locked' },
      { id: 'dyers-quarter', tier: 2, name: "Dyers' Quarter", npcName: 'The Rival Dyers', cost: { trade: 20 }, yields: { trade: 3 },
        hook: 'Dyers from three different clans all claim the quarter, and none will share the vats.', state: 'locked' },
      { id: 'astronomers-dome', tier: 2, name: "Astronomer's Dome", npcName: 'The Exiled Astronomer', cost: { trade: 15, favor: 10 }, yields: { favor: 2 },
        boon: 'Kurima gets advance warning before major sandstorms, giving the party time to prepare or shelter.',
        hook: 'An exiled court astronomer arrives, still convinced someone is trying to silence her.', state: 'locked' },
      { id: 'hanging-lantern-bazaar', tier: 2, name: 'Bazaar of Hanging Lanterns', npcName: 'The Lantern-Maker', cost: { labor: 10, trade: 20 }, yields: { trade: 4 },
        hook: 'A lantern-maker sets up shop and starts selling secrets alongside her wares.', state: 'locked' },
      { id: 'aqueduct-reservoir', tier: 2, name: 'Reservoir Aqueduct', npcName: 'The Rival Engineer', cost: { labor: 22, trade: 15 }, yields: { labor: 3 },
        boon: "Farmland becomes viable beyond the oasis proper, unlocking Kurima's growth into a true city.",
        hook: 'A rival engineer sabotages the plans before construction starts, then offers to "fix" it.', state: 'locked' },

      // --- Tier 3: Desert City ---
      { id: 'dune-throne-palace', tier: 3, name: 'Palace of the Dune Throne', npcName: 'The Dynastic Claimant', cost: { trade: 30, favor: 25 }, yields: { favor: 4 },
        boon: 'Kurima gains formal standing as a city-state, giving the party real diplomatic weight with neighboring powers.',
        hook: 'A claimant to a fallen dynasty arrives, insisting the throne &mdash; and the city &mdash; is rightfully hers.', state: 'locked' },
      { id: 'grand-exchange', tier: 3, name: 'Grand Bazaar Exchange', npcName: 'The Exchange Brokers', cost: { labor: 20, trade: 35 }, yields: { trade: 6 },
        hook: 'Exchange brokers from four rival cities arrive the same week, each certain the others are cheating.', state: 'locked' },
      { id: 'hidden-spring-temple', tier: 3, name: 'Temple of the Hidden Spring', npcName: 'The Reclusive Order', cost: { trade: 20, favor: 30 }, yields: { favor: 4 },
        boon: "The spring's blessing lets the party rest and recover fully once per long journey, even far from Kurima.",
        hook: 'A reclusive order surfaces to guard the spring &mdash; clearly they were here long before Kurima was.', state: 'locked' },
      { id: 'university-of-sands', tier: 3, name: 'University of Sands', npcName: 'The Insufferable Scholar', cost: { labor: 25, favor: 20 }, yields: { favor: 3, labor: 2 },
        boon: 'The party gains standing access to a scholar who can identify and explain desert relics they recover.',
        hook: 'A brilliant, insufferable scholar arrives to found the university, and immediately picks a fight with the temple.', state: 'locked' },
      { id: 'garrison-fortress', tier: 3, name: 'Garrison Fortress', npcName: 'The Warlord', cost: { labor: 35, trade: 20 }, yields: { labor: 3, favor: 2 },
        boon: 'Kurima becomes effectively immune to small raiding parties, freeing the party from routine defense duty.',
        hook: 'A warlord offers to garrison the fortress personally &mdash; for a price Kurima may come to regret.', state: 'locked' },
      { id: 'sand-skiff-dock', tier: 3, name: 'Sand-Skiff Dry Dock', npcName: 'The Overconfident Shipwright', cost: { labor: 15, trade: 30 }, yields: { trade: 4 },
        boon: 'The party gains a sand-skiff, cutting travel time across open desert dramatically.',
        hook: 'A shipwright with no ship-building experience insists sand-skiffs are "basically the same thing, just drier."', state: 'locked' },
      { id: 'menagerie-gardens', tier: 3, name: 'Menagerie Gardens', npcName: 'The Beast-Tamer', cost: { trade: 25, favor: 15 }, yields: { favor: 3 },
        boon: 'Exotic desert mounts &mdash; giant lizards, dune-striders &mdash; become available to the party for future journeys.',
        hook: 'A beast-tamer arrives with creatures no one in Kurima can identify, and won\'t explain where she found them.', state: 'locked' },
      { id: 'mint-assay-house', tier: 3, name: 'Mint & Assay House', npcName: 'The Suspicious Assayer', cost: { labor: 20, trade: 30 }, yields: { trade: 5 },
        hook: "An assayer arrives certain Kurima's gold has a flaw only she can detect &mdash; and profit from.", state: 'locked' },
      { id: 'aqueduct-network', tier: 3, name: 'Grand Aqueduct Network', npcName: 'The Guild Delegates', cost: { labor: 40, trade: 35, favor: 15 }, yields: { labor: 3, trade: 3, favor: 2 },
        boon: "Kurima completes its transformation from oasis to true desert metropolis, permanently unlocking its full growth.",
        hook: 'Every guild in Kurima suddenly wants credit for the aqueduct network, and several of them are lying.', state: 'locked' },
    ],
    log: [],
  };

  let state = null;
  let gmPanelOpen = false;
  let gmSelectedNpcId = '';
  const STORAGE_KEY = 'kurima-village-state';
  const CHAT_KEY = 'kurima-party-chat';
  const CHAT_NAME_KEY = 'kurima-chat-username';
  const NOTES_KEY = 'kurima-notes';
  const CHAR_KEY = 'kurima-characters';
  let chatState = { name: '', messages: [] };
  let notesState = { entries: [] };
  let selectedNoteDate = null;
  let charState = { characters: [] };
  let selectedCharId = null;
  let creatingNewChar = false;
  let unlockedCharIds = new Set();

  function simpleHash(str) {
    let hash = 5381;
    for (let i = 0; i < str.length; i++) {
      hash = ((hash << 5) + hash) + str.charCodeAt(i);
      hash = hash & hash;
    }
    return (hash >>> 0).toString(36);
  }

  function resizeImageFile(file, maxDim, quality) {
    return new Promise((resolve, reject) => {
      const reader = new FileReader();
      reader.onload = (e) => {
        const img = new Image();
        img.onload = () => {
          let { width, height } = img;
          if (width > maxDim || height > maxDim) {
            if (width > height) { height = Math.round(height * maxDim / width); width = maxDim; }
            else { width = Math.round(width * maxDim / height); height = maxDim; }
          }
          const canvas = document.createElement('canvas');
          canvas.width = width;
          canvas.height = height;
          const ctx = canvas.getContext('2d');
          ctx.drawImage(img, 0, 0, width, height);
          resolve(canvas.toDataURL('image/jpeg', quality));
        };
        img.onerror = reject;
        img.src = e.target.result;
      };
      reader.onerror = reject;
      reader.readAsDataURL(file);
    });
  }

  function escapeHtml(s) {
    return (s || '').replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;').replace(/"/g, '&quot;');
  }

  function genId(prefix) {
    return prefix + Date.now().toString(36) + Math.random().toString(36).slice(2, 6);
  }

  async function loadState() {
    try {
      const result = await storage.get(STORAGE_KEY, true);
      state = result ? JSON.parse(result.value) : structuredClone(DEFAULT_STATE);
    } catch (e) {
      state = structuredClone(DEFAULT_STATE);
    }
    render();
  }

  async function saveState() {
    try {
      await storage.set(STORAGE_KEY, JSON.stringify(state), true);
    } catch (e) {
      console.error('Could not save village state', e);
    }
  }

  function renderMessagesHtml() {
    if (chatState.messages.length === 0) return '<div class="vb-log-empty">No messages yet.</div>';
    return chatState.messages.map(m => `
      <div class="vb-chat-msg">
        <span class="vb-chat-name">${escapeHtml(m.name)}</span>
        <span class="vb-chat-text">${escapeHtml(m.text)}</span>
      </div>`).join('');
  }

  function updateChatLogDom() {
    const el = document.getElementById('vb-chat-log');
    if (!el) return;
    el.innerHTML = renderMessagesHtml();
    el.scrollTop = el.scrollHeight;
  }

  async function loadChat() {
    try {
      const result = await storage.get(CHAT_KEY, true);
      chatState.messages = result ? JSON.parse(result.value) : [];
    } catch (e) {
      chatState.messages = [];
    }
    try {
      const nameResult = await storage.get(CHAT_NAME_KEY, false);
      chatState.name = nameResult ? nameResult.value : '';
    } catch (e) {
      chatState.name = '';
    }
    updateChatLogDom();
  }

  async function refreshChat() {
    try {
      const result = await storage.get(CHAT_KEY, true);
      chatState.messages = result ? JSON.parse(result.value) : [];
      updateChatLogDom();
    } catch (e) { /* silent - just skip this refresh tick */ }
  }

  async function postChatMessage() {
    const nameInput = document.getElementById('vb-chat-name');
    const textInput = document.getElementById('vb-chat-text');
    const n = ((nameInput ? nameInput.value : '').trim()) || 'Anonymous';
    const t = (textInput ? textInput.value : '').trim();
    if (!t) return;
    try {
      const result = await storage.get(CHAT_KEY, true);
      chatState.messages = result ? JSON.parse(result.value) : [];
    } catch (e) { /* use whatever we already had in memory */ }
    chatState.messages.push({ name: n, text: t, ts: Date.now() });
    if (chatState.messages.length > 200) chatState.messages = chatState.messages.slice(-200);
    try { await storage.set(CHAT_KEY, JSON.stringify(chatState.messages), true); } catch (e) { console.error('Could not save chat message', e); }
    try { await storage.set(CHAT_NAME_KEY, n, false); } catch (e) { /* non-critical */ }
    chatState.name = n;
    if (textInput) textInput.value = '';
    updateChatLogDom();
  }

  function formatDateLabel(dateKey) {
    const d = new Date(dateKey);
    const today = new Date();
    const yest = new Date();
    yest.setDate(today.getDate() - 1);
    if (d.toDateString() === today.toDateString()) return 'Today';
    if (d.toDateString() === yest.toDateString()) return 'Yesterday';
    return d.toLocaleDateString(undefined, { month: 'short', day: 'numeric', year: 'numeric' });
  }

  function noteDateGroups() {
    const map = new Map();
    notesState.entries.forEach(e => {
      const key = new Date(e.ts).toDateString();
      if (!map.has(key)) map.set(key, []);
      map.get(key).push(e);
    });
    return Array.from(map.entries()).sort((a, b) =>
      Math.max(...b[1].map(x => x.ts)) - Math.max(...a[1].map(x => x.ts)));
  }

  function renderNotesSidebarHtml() {
    const groups = noteDateGroups();
    let html = `<div class="vb-notes-date-item ${selectedNoteDate === null ? 'active' : ''}" data-date="">All <span class="vb-notes-count">${notesState.entries.length}</span></div>`;
    groups.forEach(([dateKey, items]) => {
      html += `<div class="vb-notes-date-item ${selectedNoteDate === dateKey ? 'active' : ''}" data-date="${escapeHtml(dateKey)}">${formatDateLabel(dateKey)} <span class="vb-notes-count">${items.length}</span></div>`;
    });
    return html;
  }

  function renderNotesMainHtml() {
    let list = notesState.entries;
    if (selectedNoteDate) list = list.filter(e => new Date(e.ts).toDateString() === selectedNoteDate);
    list = [...list].sort((a, b) => b.ts - a.ts);
    if (list.length === 0) return '<div class="vb-log-empty">No notes yet.</div>';
    return list.map(e => {
      const when = new Date(e.ts).toLocaleString(undefined, { hour: 'numeric', minute: '2-digit', month: 'short', day: 'numeric' });
      const meta = `${escapeHtml(e.author || 'Anonymous')} - ${when}`;
      const body = e.type === 'image'
        ? `<img class="vb-note-image" src="${e.imageData}" alt="${escapeHtml(e.caption || '')}" />${e.caption ? `<div class="vb-note-caption">${escapeHtml(e.caption)}</div>` : ''}`
        : `<div class="vb-note-text">${escapeHtml(e.text)}</div>`;
      return `<div class="vb-note-card">
        ${body}
        <div class="vb-note-meta">${meta}</div>
        <button class="vb-note-delete" data-action="note-delete" data-id="${e.id}" title="Delete">&#10005;</button>
      </div>`;
    }).join('');
  }

  function bindNotesInteractions(scope) {
    scope.querySelectorAll('.vb-notes-date-item').forEach(el =>
      el.addEventListener('click', () => selectNoteDate(el.dataset.date || null)));
    scope.querySelectorAll('[data-action="note-delete"]').forEach(btn =>
      btn.addEventListener('click', () => deleteNoteEntry(btn.dataset.id)));
  }

  function updateNotesDom() {
    const sidebarEl = document.getElementById('vb-notes-sidebar');
    const mainEl = document.getElementById('vb-notes-main');
    if (sidebarEl) sidebarEl.innerHTML = renderNotesSidebarHtml();
    if (mainEl) mainEl.innerHTML = renderNotesMainHtml();
    if (sidebarEl) bindNotesInteractions(sidebarEl);
    if (mainEl) bindNotesInteractions(mainEl);
  }

  function selectNoteDate(dateKey) {
    selectedNoteDate = dateKey || null;
    updateNotesDom();
  }

  async function loadNotes() {
    try {
      const result = await storage.get(NOTES_KEY, true);
      notesState.entries = result ? JSON.parse(result.value) : [];
    } catch (e) {
      notesState.entries = [];
    }
    updateNotesDom();
  }

  async function refreshNotes() {
    try {
      const result = await storage.get(NOTES_KEY, true);
      notesState.entries = result ? JSON.parse(result.value) : [];
      updateNotesDom();
    } catch (e) { /* silent - just skip this refresh tick */ }
  }

  async function saveNotes() {
    try {
      await storage.set(NOTES_KEY, JSON.stringify(notesState.entries), true);
    } catch (e) {
      console.error('Could not save notes', e);
      throw e;
    }
  }

  async function addTextNote() {
    const textInput = document.getElementById('vb-note-text');
    const authorInput = document.getElementById('vb-note-author');
    const t = (textInput ? textInput.value : '').trim();
    if (!t) return;
    const author = (authorInput ? authorInput.value : '').trim() || chatState.name || 'Anonymous';
    try {
      const result = await storage.get(NOTES_KEY, true);
      notesState.entries = result ? JSON.parse(result.value) : [];
    } catch (e) { /* use whatever we already had in memory */ }
    notesState.entries.push({ id: genId('n'), type: 'text', author, text: t, ts: Date.now() });
    try {
      await saveNotes();
      if (textInput) textInput.value = '';
      updateNotesDom();
    } catch (e) {
      alert('Could not save that note - please try again.');
    }
  }

  async function addImageNote() {
    const fileInput = document.getElementById('vb-note-image');
    const captionInput = document.getElementById('vb-note-caption');
    const authorInput = document.getElementById('vb-note-author');
    const addBtn = document.querySelector('[data-action="note-add-image"]');
    if (!fileInput || !fileInput.files || !fileInput.files[0]) return;
    const file = fileInput.files[0];
    const author = (authorInput ? authorInput.value : '').trim() || chatState.name || 'Anonymous';
    const caption = (captionInput ? captionInput.value : '').trim();
    if (addBtn) { addBtn.disabled = true; addBtn.textContent = 'Uploading...'; }
    try {
      let dataUrl = await resizeImageFile(file, 800, 0.7);
      if (dataUrl.length > 900000) dataUrl = await resizeImageFile(file, 600, 0.5);
      try {
        const result = await storage.get(NOTES_KEY, true);
        notesState.entries = result ? JSON.parse(result.value) : [];
      } catch (e) { /* use whatever we already had in memory */ }
      notesState.entries.push({ id: genId('n'), type: 'image', author, caption, imageData: dataUrl, ts: Date.now() });
      await saveNotes();
      fileInput.value = '';
      if (captionInput) captionInput.value = '';
      updateNotesDom();
    } catch (e) {
      console.error('Could not add image note', e);
      alert('Could not save that image - it may be too large. Try a smaller picture.');
    } finally {
      if (addBtn) { addBtn.disabled = false; addBtn.textContent = 'Add Image'; }
    }
  }

  async function deleteNoteEntry(id) {
    try {
      const result = await storage.get(NOTES_KEY, true);
      notesState.entries = result ? JSON.parse(result.value) : notesState.entries;
    } catch (e) { /* use whatever we already had in memory */ }
    notesState.entries = notesState.entries.filter(e => e.id !== id);
    try {
      await saveNotes();
    } catch (e) { /* already logged in saveNotes */ }
    updateNotesDom();
  }

  // ---- Character sheets ----

  function trackHtml(field, label, max, marked, editable, charId) {
    let pips = '';
    for (let i = 0; i < max; i++) {
      const filled = i < marked;
      pips += `<span class="vb-pip ${filled ? 'filled' : ''} ${editable ? 'clickable' : ''}" ${editable ? `data-action="pip-toggle" data-field="${field}" data-index="${i}" data-charid="${charId || ''}"` : ''}></span>`;
    }
    const maxControl = editable ? `<input class="vb-track-max" id="cf-${field}-max" type="number" min="1" value="${max}" />` : '';
    return `<div class="vb-track">
      <div class="vb-track-label">${label} ${maxControl}</div>
      <div class="vb-pip-track">${pips}</div>
    </div>`;
  }

  function charFieldsHtml(c, editable, isNew) {
    const val = (v) => (v === undefined || v === null) ? '' : v;
    const field = (id, label, value, type) => editable
      ? `<label class="vb-cf-label">${label}<input id="${id}" type="${type || 'text'}" value="${escapeHtml(String(val(value)))}" /></label>`
      : `<div class="vb-cf-view"><span class="vb-cf-view-label">${label}</span><span class="vb-cf-view-value">${escapeHtml(String(val(value))) || '&mdash;'}</span></div>`;
    const area = (id, label, value) => editable
      ? `<label class="vb-cf-label vb-cf-label-wide">${label}<textarea id="${id}">${escapeHtml(val(value))}</textarea></label>`
      : `<div class="vb-cf-view vb-cf-view-wide"><span class="vb-cf-view-label">${label}</span><div class="vb-cf-view-text">${escapeHtml(val(value)) || '&mdash;'}</div></div>`;

    const t = c.traits || {};
    const w1 = c.weapon1 || {};
    const w2 = c.weapon2 || {};

    const tracksHtml = isNew
      ? `<div class="vb-cf-row">
          <label class="vb-cf-label">Max HP<input id="cf-hp-max" type="number" min="1" value="${val(c.hpMax) || 6}" /></label>
          <label class="vb-cf-label">Max Stress<input id="cf-stress-max" type="number" min="1" value="${val(c.stressMax) || 6}" /></label>
          <label class="vb-cf-label">Max Hope<input id="cf-hope-max" type="number" min="1" value="${val(c.hopeMax) || 6}" /></label>
        </div>`
      : `<div class="vb-cf-tracks">
          ${trackHtml('hp', 'HP', c.hpMax || 6, c.hpMarked || 0, editable, c.id)}
          ${trackHtml('stress', 'Stress', c.stressMax || 6, c.stressMarked || 0, editable, c.id)}
          ${trackHtml('hope', 'Hope', c.hopeMax || 6, c.hopeMarked || 0, editable, c.id)}
        </div>`;

    return `
      <div class="vb-cf-row">
        ${field('cf-charname', 'Character Name', c.characterName)}
        ${field('cf-level', 'Level', c.level, 'number')}
        ${field('cf-class', 'Class', c.className)}
        ${field('cf-subclass', 'Subclass', c.subclassName)}
        ${field('cf-ancestry', 'Ancestry', c.ancestry)}
        ${field('cf-community', 'Community', c.community)}
      </div>
      <div class="vb-cf-row">
        ${field('cf-agility', 'Agility', t.agility, 'number')}
        ${field('cf-strength', 'Strength', t.strength, 'number')}
        ${field('cf-finesse', 'Finesse', t.finesse, 'number')}
        ${field('cf-instinct', 'Instinct', t.instinct, 'number')}
        ${field('cf-presence', 'Presence', t.presence, 'number')}
        ${field('cf-knowledge', 'Knowledge', t.knowledge, 'number')}
      </div>
      <div class="vb-cf-row">
        ${field('cf-evasion', 'Evasion', c.evasion, 'number')}
        ${field('cf-armor', 'Armor Score', c.armorScore, 'number')}
        ${field('cf-prof', 'Proficiency', c.proficiency, 'number')}
        ${field('cf-gold', 'Gold', c.gold)}
      </div>
      ${tracksHtml}
      <div class="vb-cf-row">
        ${field('cf-w1-name', 'Primary Weapon', w1.name)}
        ${field('cf-w1-trait', 'Trait', w1.trait)}
        ${field('cf-w1-range', 'Range', w1.range)}
        ${field('cf-w1-damage', 'Damage', w1.damage)}
      </div>
      <div class="vb-cf-row">
        ${field('cf-w2-name', 'Secondary Weapon', w2.name)}
        ${field('cf-w2-trait', 'Trait', w2.trait)}
        ${field('cf-w2-range', 'Range', w2.range)}
        ${field('cf-w2-damage', 'Damage', w2.damage)}
      </div>
      ${area('cf-domain', 'Domain Cards', c.domainCards)}
      ${renderDomainCardsPickerHtml(editable)}
      ${area('cf-experiences', 'Experiences', c.experiences)}
      ${area('cf-features', 'Features & Notes', c.features)}
      ${area('cf-background', 'Background & Connections', c.background)}
      ${area('cf-inventory', 'Inventory', c.inventory)}
    `;
  }

  function renderCharSidebarHtml() {
    let html = `<div class="vb-char-item ${creatingNewChar ? 'active' : ''}" data-action="char-new">+ New Character</div>`;
    charState.characters.forEach(c => {
      const active = !creatingNewChar && selectedCharId === c.id;
      html += `<div class="vb-char-item ${active ? 'active' : ''}" data-action="char-select" data-id="${c.id}">
        <div class="vb-char-item-name">${escapeHtml(c.characterName || 'Unnamed')}</div>
        <div class="vb-char-item-owner">${escapeHtml(c.ownerName || '')}</div>
      </div>`;
    });
    return html;
  }

  function renderCharMainHtml() {
    if (creatingNewChar) {
      return `
        <div class="vb-char-form">
          <div class="vb-cf-row">
            <label class="vb-cf-label">Your Name<input id="cf-owner" type="text" /></label>
            <label class="vb-cf-label">Set a Password<input id="cf-password" type="password" /></label>
          </div>
          ${charFieldsHtml({ hpMax: 6, stressMax: 6, hopeMax: 6 }, true, true)}
          <button class="vb-char-save-btn" data-action="char-create">Create Character</button>
        </div>`;
    }
    const c = charState.characters.find(x => x.id === selectedCharId);
    if (!c) return '<div class="vb-log-empty">Select a character on the left, or create a new one.</div>';
    const unlocked = unlockedCharIds.has(c.id);
    if (!unlocked) {
      return `
        <div class="vb-char-form">
          <div class="vb-char-view-header">${escapeHtml(c.characterName || 'Unnamed')} <span class="vb-char-view-owner">played by ${escapeHtml(c.ownerName || '')}</span></div>
          ${charFieldsHtml(c, false, false)}
          <div class="vb-char-unlock-row">
            <input id="cf-unlock-password" type="password" placeholder="Password to edit" />
            <button data-action="char-unlock" data-id="${c.id}">Unlock to Edit</button>
          </div>
        </div>`;
    }
    return `
      <div class="vb-char-form">
        <div class="vb-char-view-header">Editing ${escapeHtml(c.characterName || 'Unnamed')} <button class="vb-char-lock-btn" data-action="char-lock" data-id="${c.id}">Lock</button></div>
        ${charFieldsHtml(c, true, false)}
        <button class="vb-char-save-btn" data-action="char-save" data-id="${c.id}">Save Changes</button>
      </div>`;
  }

  function bindCharInteractions(scope) {
    scope.querySelectorAll('[data-action="char-select"]').forEach(x =>
      x.addEventListener('click', () => selectCharacter(x.dataset.id)));
    scope.querySelectorAll('[data-action="char-new"]').forEach(x =>
      x.addEventListener('click', startNewCharacter));
    scope.querySelectorAll('[data-action="char-create"]').forEach(x =>
      x.addEventListener('click', createCharacter));
    scope.querySelectorAll('[data-action="char-unlock"]').forEach(x =>
      x.addEventListener('click', () => unlockCharacter(x.dataset.id)));
    scope.querySelectorAll('[data-action="char-lock"]').forEach(x =>
      x.addEventListener('click', () => lockCharacter(x.dataset.id)));
    scope.querySelectorAll('[data-action="char-save"]').forEach(x =>
      x.addEventListener('click', () => saveCharacterEdits(x.dataset.id)));
    scope.querySelectorAll('[data-action="pip-toggle"]').forEach(x =>
      x.addEventListener('click', () => togglePip(x.dataset.charid, x.dataset.field, parseInt(x.dataset.index, 10))));
    scope.querySelectorAll('[data-action="domain-chip-insert"]').forEach(x =>
      x.addEventListener('click', () => insertDomainCard(x.dataset.name)));
  }

  function insertDomainCard(name) {
    const textarea = document.getElementById('cf-domain');
    if (!textarea) return;
    const lines = textarea.value.split('\n').map(s => s.trim()).filter(Boolean);
    if (lines.includes(name)) return;
    textarea.value = lines.concat([name]).join('\n');
  }

  function updateCharSidebarDom() {
    const el = document.getElementById('vb-char-sidebar');
    if (!el) return;
    el.innerHTML = renderCharSidebarHtml();
    bindCharInteractions(el);
  }

  function updateCharMainDom() {
    const el = document.getElementById('vb-char-main');
    if (!el) return;
    el.innerHTML = renderCharMainHtml();
    bindCharInteractions(el);
  }

  function selectCharacter(id) {
    creatingNewChar = false;
    selectedCharId = id;
    updateCharSidebarDom();
    updateCharMainDom();
  }

  function startNewCharacter() {
    creatingNewChar = true;
    selectedCharId = null;
    updateCharSidebarDom();
    updateCharMainDom();
  }

  function unlockCharacter(id) {
    const passInput = document.getElementById('cf-unlock-password');
    const entered = passInput ? passInput.value : '';
    const c = charState.characters.find(x => x.id === id);
    if (!c) return;
    if (simpleHash(entered) === c.passwordHash) {
      unlockedCharIds.add(id);
      updateCharMainDom();
    } else {
      alert('Incorrect password.');
    }
  }

  function lockCharacter(id) {
    unlockedCharIds.delete(id);
    updateCharMainDom();
  }

  function collectCharFieldsFromForm(base, isNew) {
    const get = (id) => { const el = document.getElementById(id); return el ? el.value : ''; };
    const num = (id, fallback) => { const v = parseInt(get(id), 10); return isNaN(v) ? (fallback || 0) : v; };
    const c = Object.assign({}, base);
    c.characterName = get('cf-charname');
    c.level = num('cf-level', 1);
    c.className = get('cf-class');
    c.subclassName = get('cf-subclass');
    c.ancestry = get('cf-ancestry');
    c.community = get('cf-community');
    c.traits = {
      agility: num('cf-agility'), strength: num('cf-strength'), finesse: num('cf-finesse'),
      instinct: num('cf-instinct'), presence: num('cf-presence'), knowledge: num('cf-knowledge')
    };
    c.evasion = num('cf-evasion');
    c.armorScore = num('cf-armor');
    c.proficiency = num('cf-prof');
    c.gold = get('cf-gold');
    c.weapon1 = { name: get('cf-w1-name'), trait: get('cf-w1-trait'), range: get('cf-w1-range'), damage: get('cf-w1-damage') };
    c.weapon2 = { name: get('cf-w2-name'), trait: get('cf-w2-trait'), range: get('cf-w2-range'), damage: get('cf-w2-damage') };
    c.domainCards = get('cf-domain');
    c.experiences = get('cf-experiences');
    c.features = get('cf-features');
    c.background = get('cf-background');
    c.inventory = get('cf-inventory');
    if (isNew) {
      c.hpMax = num('cf-hp-max', 6); c.hpMarked = 0;
      c.stressMax = num('cf-stress-max', 6); c.stressMarked = 0;
      c.hopeMax = num('cf-hope-max', 6); c.hopeMarked = 0;
    } else {
      c.hpMax = num('cf-hp-max', base.hpMax || 6);
      c.stressMax = num('cf-stress-max', base.stressMax || 6);
      c.hopeMax = num('cf-hope-max', base.hopeMax || 6);
      c.hpMarked = Math.min(base.hpMarked || 0, c.hpMax);
      c.stressMarked = Math.min(base.stressMarked || 0, c.stressMax);
      c.hopeMarked = Math.min(base.hopeMarked || 0, c.hopeMax);
    }
    c.updatedAt = Date.now();
    return c;
  }

  async function createCharacter() {
    const ownerInput = document.getElementById('cf-owner');
    const passInput = document.getElementById('cf-password');
    const nameInput = document.getElementById('cf-charname');
    const owner = (ownerInput ? ownerInput.value : '').trim();
    const pass = passInput ? passInput.value : '';
    const name = (nameInput ? nameInput.value : '').trim();
    if (!owner || !pass || !name) {
      alert('Please enter your name, a password, and a character name before creating.');
      return;
    }
    const newChar = collectCharFieldsFromForm({ id: genId('c'), ownerName: owner, passwordHash: simpleHash(pass) }, true);
    try {
      const result = await storage.get(CHAR_KEY, true);
      charState.characters = result ? JSON.parse(result.value) : [];
    } catch (e) { /* use whatever we already had in memory */ }
    charState.characters.push(newChar);
    try {
      await storage.set(CHAR_KEY, JSON.stringify(charState.characters), true);
    } catch (e) {
      alert('Could not save the character - please try again.');
      return;
    }
    unlockedCharIds.add(newChar.id);
    creatingNewChar = false;
    selectedCharId = newChar.id;
    updateCharSidebarDom();
    updateCharMainDom();
  }

  async function saveCharacterEdits(id) {
    const idx = charState.characters.findIndex(x => x.id === id);
    if (idx === -1) return;
    const existing = charState.characters[idx];
    const updated = collectCharFieldsFromForm(existing, false);
    updated.id = existing.id;
    updated.ownerName = existing.ownerName;
    updated.passwordHash = existing.passwordHash;
    try {
      const result = await storage.get(CHAR_KEY, true);
      charState.characters = result ? JSON.parse(result.value) : charState.characters;
    } catch (e) { /* use whatever we already had in memory */ }
    const idx2 = charState.characters.findIndex(x => x.id === id);
    if (idx2 !== -1) charState.characters[idx2] = updated; else charState.characters.push(updated);
    try {
      await storage.set(CHAR_KEY, JSON.stringify(charState.characters), true);
    } catch (e) {
      alert('Could not save changes - please try again.');
      return;
    }
    updateCharSidebarDom();
    updateCharMainDom();
  }

  function togglePip(charId, field, index) {
    if (!unlockedCharIds.has(charId)) return;
    const c = charState.characters.find(x => x.id === charId);
    if (!c) return;
    const markedKey = field + 'Marked';
    const current = c[markedKey] || 0;
    c[markedKey] = (current === index + 1) ? index : index + 1;
    storage.set(CHAR_KEY, JSON.stringify(charState.characters), true).catch(e => console.error('Could not save track update', e));
    updateCharMainDom();
  }

  function renderDomainCardsPickerHtml(editable) {
    const domainKeys = Object.keys(DOMAIN_LIBRARY);
    if (domainKeys.length === 0) return '';
    const blocks = domainKeys.map(key => {
      const d = DOMAIN_LIBRARY[key];
      const levelGroups = d.cards.map(lvl => `
        <div class="vb-domain-level-group">
          <div class="vb-domain-level-label">Level ${lvl.level}</div>
          <div class="vb-domain-card-row">
            ${lvl.options.map(name => `<span class="vb-domain-card-chip ${editable ? 'clickable' : ''}" ${editable ? `data-action="domain-chip-insert" data-name="${escapeHtml(name)}" title="Add to Domain Cards"` : ''}>${escapeHtml(name)}</span>`).join('')}
          </div>
        </div>`).join('');
      return `
        <div class="vb-domain-block-nested">
          <div class="vb-domain-title-nested">${escapeHtml(d.name)}</div>
          ${levelGroups}
        </div>`;
    }).join('');
    return `<details class="vb-domain-picker">
      <summary>Browse Domain Cards</summary>
      ${blocks}
    </details>`;
  }

  async function loadCharacters() {
    try {
      const result = await storage.get(CHAR_KEY, true);
      charState.characters = result ? JSON.parse(result.value) : [];
    } catch (e) {
      charState.characters = [];
    }
    updateCharSidebarDom();
    updateCharMainDom();
  }

  async function refreshCharacters() {
    try {
      const result = await storage.get(CHAR_KEY, true);
      charState.characters = result ? JSON.parse(result.value) : [];
      updateCharSidebarDom();
      if (!(selectedCharId && unlockedCharIds.has(selectedCharId))) {
        updateCharMainDom();
      }
    } catch (e) { /* silent - just skip this refresh tick */ }
  }

  function tierPrereqMet(tier) {
    if (tier === 1) return true;
    const priorTier = tier - 1;
    const builtInPrior = state.buildings.filter(b => b.tier === priorTier && b.state === 'built').length;
    return builtInPrior >= 2;
  }

  function refreshLocks() {
    state.buildings.forEach(b => {
      if (b.state === 'built') return;
      b.state = tierPrereqMet(b.tier) ? 'available' : 'locked';
    });
  }

  function canAfford(cost) {
    return Object.entries(cost).every(([k, v]) => state.resources[k] >= v);
  }

  function costString(cost) {
    return Object.entries(cost).map(([k, v]) =>
      `<span class="${state.resources[k] >= v ? 'met' : 'unmet'}">${v} ${RESOURCE_META[k].label}</span>`
    ).join('');
  }
  function yieldString(y) {
    return '+' + Object.entries(y).map(([k, v]) => `${v} ${RESOURCE_META[k].label}`).join(', ') + ' / turn';
  }

  function constructBuilding(id) {
    const b = state.buildings.find(x => x.id === id);
    if (!b || b.state !== 'available' || !canAfford(b.cost)) return;
    Object.entries(b.cost).forEach(([k, v]) => state.resources[k] -= v);
    Object.entries(b.yields).forEach(([k, v]) => state.income[k] = (state.income[k] || 0) + v);
    b.state = 'built';
    b.justBuilt = true;
    state.log.unshift({ type: 'arrival', id: genId('a'), source: b.name, text: b.hook, boon: b.boon || null });
    refreshLocks();
    saveState();
    render();
    setTimeout(() => { b.justBuilt = false; }, 800);
  }

  function tickIncome() {
    Object.keys(state.income).forEach(k => {
      state.resources[k] = (state.resources[k] || 0) + (state.income[k] || 0);
    });
    saveState();
    render();
  }

  function nudgeResource(key, delta) {
    state.resources[key] = Math.max(0, (state.resources[key] || 0) + delta);
    refreshLocks();
    saveState();
    render();
  }

  function addQuestEntry(text) {
    const trimmed = (text || '').trim();
    if (!trimmed) return;
    state.log.unshift({ type: 'quest', text: trimmed, id: genId('q') });
    saveState();
    render();
  }

  function deleteLogEntry(id) {
    state.log = state.log.filter(l => l.id !== id);
    saveState();
    render();
  }

  function updateLogField(id, field, value) {
    const entry = state.log.find(l => l.id === id);
    if (entry) { entry[field] = value; saveState(); }
  }

  function getNpcRoster() {
    const pending = state.buildings
      .filter(b => b.state !== 'built')
      .map(b => ({ key: 'b:' + b.id, name: b.npcName || b.name, text: b.hook, group: 'Awaiting construction', tag: b.name }));
    const arrived = state.log
      .filter(l => l.type === 'arrival')
      .map(l => ({ key: 'l:' + l.id, name: l.source, text: l.text, group: 'Already arrived' }));
    return { pending, arrived };
  }

  function findRosterEntry(key) {
    if (!key) return null;
    if (key.startsWith('b:')) {
      const b = state.buildings.find(x => x.id === key.slice(2));
      return b ? { name: b.npcName || b.name, text: b.hook } : null;
    }
    if (key.startsWith('l:')) {
      const l = state.log.find(x => x.id === key.slice(2));
      return l ? { name: l.source, text: l.text } : null;
    }
    return null;
  }

  function addGmArrival(source, text) {
    const s = (source || '').trim();
    const t = (text || '').trim();
    if (!t) return;
    state.log.unshift({ type: 'arrival', id: genId('a'), source: s || 'Unnamed', text: t, boon: null });
    gmSelectedNpcId = '';
    saveState();
    render();
  }

  function selectGmNpc(key) {
    gmSelectedNpcId = key;
    render();
  }

  function saveGmNpcEdits(source, text) {
    if (!gmSelectedNpcId) return;
    const s = (source || '').trim();
    const t = (text || '').trim();
    if (gmSelectedNpcId.startsWith('b:')) {
      const b = state.buildings.find(x => x.id === gmSelectedNpcId.slice(2));
      if (b) { b.npcName = s || b.npcName; b.hook = t || b.hook; }
    } else if (gmSelectedNpcId.startsWith('l:')) {
      const l = state.log.find(x => x.id === gmSelectedNpcId.slice(2));
      if (l) { l.source = s || l.source; l.text = t || l.text; }
    }
    saveState();
    render();
  }

  function toggleGmPanel() {
    gmPanelOpen = !gmPanelOpen;
    render();
  }

  function renderSkyline() {
    // Procedural silhouette standing in for future map/skyline art.
    // Each built building adds a shape; swap this <svg> for an <img>/layered
    // art asset later without touching the rest of the layout.
    const built = state.buildings.filter(b => b.state === 'built');
    const w = 1100, h = 180;
    let shapes = '';
    const baseY = 150;
    built.forEach((b, i) => {
      const x = 30 + i * ((w - 60) / Math.max(1, built.length));
      const bh = 26 + (b.tier * 16);
      const bw = 30;
      const color = b.tier === 1 ? 'rgba(177,72,31,0.55)' : b.tier === 2 ? 'rgba(31,122,108,0.5)' : 'rgba(199,154,62,0.6)';
      // dome-topped silhouette rather than a flat gable, to read as desert architecture
      shapes += `<rect x="${x}" y="${baseY - bh}" width="${bw}" height="${bh}" fill="${color}" rx="2"></rect>`;
      shapes += `<circle cx="${x + bw/2}" cy="${baseY - bh}" r="${bw/2 - 2}" fill="${color}"></circle>`;
    });
    return `<svg id="vb-skyline-art" viewBox="0 0 ${w} ${h}" preserveAspectRatio="none">
      <rect x="0" y="${baseY}" width="${w}" height="${h - baseY}" fill="rgba(23,18,37,0.65)"></rect>
      ${shapes}
    </svg>`;
  }

  function render() {
    if (!state) return;
    refreshLocks();

    const tiers = [1, 2, 3];
    const tierNames = { 1: 'Oasis Camp', 2: 'Trade Town', 3: 'Desert City' };

    root.innerHTML = `
      <div class="vb-header">
        <div>
          <input class="vb-title" value="${state.villageName}" placeholder="Settlement name" />
        </div>
        <div class="vb-subtitle">${state.tagline}</div>
        <button class="vb-gm-toggle ${gmPanelOpen ? 'active' : ''}" data-action="toggle-gm">${gmPanelOpen ? 'Hide GM Panel' : 'GM Panel'}</button>
      </div>

      <div class="vb-hero">
        ${renderSkyline()}
        <div class="vb-hero-caption">skyline grows as you build</div>
      </div>

      <div class="vb-grid">
        <div>
          <div class="vb-panel">
            <h3>Resources</h3>
            ${Object.keys(RESOURCE_META).map(key => {
              const meta = RESOURCE_META[key];
              const val = state.resources[key] || 0;
              const pct = Math.min(100, (val / GAUGE_CAP) * 100);
              const inc = state.income[key] || 0;
              return `
              <div class="vb-resource">
                <div class="vb-resource-row">
                  <span class="vb-resource-name">${meta.label}</span>
                  <span class="vb-resource-controls">
                    <button class="vb-step-btn" data-action="step" data-key="${key}" data-delta="-1" title="-1 ${meta.label}">&minus;</button>
                    <span class="vb-resource-val">${val}</span>
                    <button class="vb-step-btn" data-action="step" data-key="${key}" data-delta="1" title="+1 ${meta.label}">+</button>
                  </span>
                </div>
                <div class="vb-gauge-track">
                  <div class="vb-gauge-fill ${meta.cls}" style="width:${pct}%"></div>
                </div>
                <div class="vb-resource-income">${inc > 0 ? '+' + inc + ' / turn' : 'no income yet'}</div>
              </div>`;
            }).join('')}
            <div class="vb-add-row">
              <button data-action="tick">Advance a turn</button>
            </div>
            <div class="vb-add-row">
              <button data-action="add" data-key="labor">+5 Labor</button>
              <button data-action="add" data-key="trade">+5 Trade</button>
              <button data-action="add" data-key="favor">+5 Favor</button>
            </div>
          </div>

          <div class="vb-panel">
            <h3>Jobs, Rumors, and Arrivals</h3>
            <div class="vb-post-form">
              <textarea id="vb-post-input" placeholder="Write a job, rumor, or quest for the board..."></textarea>
              <button class="vb-post-btn" data-action="post">Post to the Board</button>
            </div>
            ${state.log.filter(l => l.type === 'quest').map(l => `
              <div class="vb-quest-card">
                <div class="vb-quest-label">posted to the board</div>
                <div class="vb-quest-text">${l.text}</div>
                <button class="vb-quest-remove" data-action="delete-log" data-id="${l.id}">take down</button>
              </div>`).join('')}
            <div class="vb-log">
              ${state.log.filter(l => l.type !== 'quest').length === 0
                ? '<div class="vb-log-empty">Nothing stirring yet &mdash; build something.</div>'
                : state.log.filter(l => l.type !== 'quest').map(l => `
                  <div class="vb-log-entry">
                    <span class="vb-log-source">${l.source}</span>${l.text}
                  </div>`).join('')}
            </div>
          </div>

          <div class="vb-panel">
            <h3>Party Chat</h3>
            <div class="vb-chat-note">Shared &mdash; visible to everyone using this dashboard.</div>
            <div class="vb-chat-log" id="vb-chat-log">${renderMessagesHtml()}</div>
            <div class="vb-chat-form">
              <input id="vb-chat-name" placeholder="Your name" value="${escapeHtml(chatState.name)}" />
              <div class="vb-chat-input-row">
                <input id="vb-chat-text" placeholder="Type a message..." />
                <button data-action="chat-post">Send</button>
              </div>
            </div>
          </div>
        </div>

        <div class="vb-panel">
          <h3>The Kurima Building Plan</h3>
          ${tiers.map(t => {
            const inTier = state.buildings.filter(b => b.tier === t);
            const unlocked = t === 1 || tierPrereqMet(t);
            if (!unlocked) {
              return `
              <div class="vb-tier">
                <div class="vb-tier-label"><span>${tierNames[t]}</span><span class="vb-tier-line"></span></div>
                <div class="vb-tier-teaser">&#128274; Undiscovered &mdash; grow the tier above to reveal what's next.</div>
              </div>`;
            }
            return `
            <div class="vb-tier">
              <div class="vb-tier-label"><span>${tierNames[t]}</span><span class="vb-tier-line"></span></div>
              <div class="vb-cards">
                ${inTier.map(b => {
                  const affordable = b.state === 'available' && canAfford(b.cost);
                  const statusText = b.state === 'built' ? 'Built' : b.state === 'locked' ? 'Locked' : 'Available';
                  return `
                  <div class="vb-card ${b.state} ${affordable ? 'affordable' : ''} ${b.justBuilt ? 'just-built' : ''}">
                    <div class="vb-card-name" contenteditable="true" data-edit="name" data-id="${b.id}">${b.name}</div>
                    <div class="vb-card-status">${statusText}</div>
                    ${b.state !== 'built' ? `<div class="vb-card-cost">${costString(b.cost)}</div>` : ''}
                    <div class="vb-card-yield">${yieldString(b.yields)}</div>
                    ${b.boon ? `<div class="vb-card-boon">&#10022; ${b.boon}</div>` : ''}
                    <button class="vb-card-btn" data-action="build" data-id="${b.id}"
                      ${b.state === 'built' || !affordable ? 'disabled' : ''}>
                      ${b.state === 'built' ? 'Complete' : b.state === 'locked' ? 'Locked' : 'Build'}
                    </button>
                  </div>`;
                }).join('')}
              </div>
            </div>`;
          }).join('')}
        </div>
      </div>

      <div class="vb-panel">
        <h3>Notes &amp; Images</h3>
        <div class="vb-notes-note">Shared &mdash; visible to everyone using this dashboard.</div>
        <div class="vb-notes-composer">
          <textarea id="vb-note-text" placeholder="Write a note..."></textarea>
          <div class="vb-notes-composer-row">
            <input id="vb-note-author" placeholder="Your name (optional)" value="${escapeHtml(chatState.name)}" />
            <button data-action="note-add-text">Add Note</button>
          </div>
          <div class="vb-notes-composer-row">
            <input type="file" id="vb-note-image" accept="image/*" />
            <input id="vb-note-caption" placeholder="Caption (optional)" />
            <button data-action="note-add-image">Add Image</button>
          </div>
        </div>
        <div class="vb-notes-body">
          <div class="vb-notes-sidebar" id="vb-notes-sidebar">${renderNotesSidebarHtml()}</div>
          <div class="vb-notes-main" id="vb-notes-main">${renderNotesMainHtml()}</div>
        </div>
      </div>

      <div class="vb-panel">
        <h3>Character Sheets</h3>
        <div class="vb-char-password-note">Shared &mdash; every player can view all sheets. Creating one sets a password only you need to know to edit it later. This is a lightweight courtesy lock, not real security &mdash; anyone determined enough could bypass it, so don't use a password you rely on elsewhere.</div>
        <div class="vb-char-body">
          <div class="vb-char-sidebar" id="vb-char-sidebar">${renderCharSidebarHtml()}</div>
          <div class="vb-char-main" id="vb-char-main">${renderCharMainHtml()}</div>
        </div>
      </div>

      ${gmPanelOpen ? `
      <div class="vb-gm-panel">
        <div class="vb-gm-panel-label">GM Panel &mdash; hidden from players</div>
        <div class="vb-gm-panel-sub">Review and edit every NPC, rumor, and posted quest. Close this before sharing your screen.</div>
        ${state.log.length === 0 ? '<div class="vb-gm-empty">Nothing logged yet.</div>' : state.log.map(l => `
          <div class="vb-gm-row">
            ${l.type === 'arrival'
              ? `<input data-gm-field="source" data-id="${l.id}" value="${(l.source || '').replace(/"/g, '&quot;')}" placeholder="NPC name" />`
              : `<span class="vb-gm-tag">posted quest</span>`}
            <textarea data-gm-field="text" data-id="${l.id}">${l.text}</textarea>
            <button class="vb-gm-delete" data-action="delete-log" data-id="${l.id}" title="Delete">&#10005;</button>
          </div>`).join('')}
        <div class="vb-gm-add">
          <input id="vb-gm-add-source" placeholder="NPC name" value="${(() => {
            const sel = findRosterEntry(gmSelectedNpcId);
            return sel ? (sel.name || '').replace(/"/g, '&quot;') : '';
          })()}" />
          <select id="vb-gm-add-select">
            <option value="">&mdash; New NPC &mdash;</option>
            ${(() => {
              const { pending, arrived } = getNpcRoster();
              const opt = (n) => `<option value="${n.key}" ${n.key === gmSelectedNpcId ? 'selected' : ''}>${(n.name || 'Unnamed').replace(/</g, '&lt;')}${n.tag ? ' &mdash; ' + n.tag.replace(/</g, '&lt;') : ''}</option>`;
              return (pending.length ? `<optgroup label="Awaiting construction">${pending.map(opt).join('')}</optgroup>` : '')
                + (arrived.length ? `<optgroup label="Already arrived">${arrived.map(opt).join('')}</optgroup>` : '');
            })()}
          </select>
          <textarea id="vb-gm-add-text" placeholder="Rumor or hook text...">${(() => {
            const sel = findRosterEntry(gmSelectedNpcId);
            return sel ? sel.text : '';
          })()}</textarea>
          <button class="vb-gm-add-btn" data-action="gm-add-arrival">Add NPC</button>
          <button class="vb-gm-save-btn" data-action="gm-save-arrival">Save Changes</button>
        </div>
      </div>` : ''}

      <div class="vb-footnote">Progress saves automatically to your browser &mdash; reopen this artifact anytime to pick up where you left off.</div>
    `;

    root.querySelectorAll('[data-action="build"]').forEach(btn =>
      btn.addEventListener('click', () => constructBuilding(btn.dataset.id)));
    root.querySelectorAll('[data-action="tick"]').forEach(btn =>
      btn.addEventListener('click', tickIncome));
    root.querySelectorAll('[data-action="add"]').forEach(btn =>
      btn.addEventListener('click', () => nudgeResource(btn.dataset.key, 5)));
    root.querySelectorAll('[data-action="step"]').forEach(btn =>
      btn.addEventListener('click', () => nudgeResource(btn.dataset.key, parseInt(btn.dataset.delta, 10))));
    root.querySelectorAll('[data-action="post"]').forEach(btn =>
      btn.addEventListener('click', () => {
        const input = root.querySelector('#vb-post-input');
        addQuestEntry(input ? input.value : '');
      }));
    root.querySelectorAll('[data-action="remove-quest"], [data-action="delete-log"]').forEach(btn =>
      btn.addEventListener('click', () => deleteLogEntry(btn.dataset.id)));
    root.querySelectorAll('[data-action="toggle-gm"]').forEach(btn =>
      btn.addEventListener('click', toggleGmPanel));
    root.querySelectorAll('[data-gm-field]').forEach(el =>
      el.addEventListener('blur', () => updateLogField(el.dataset.id, el.dataset.gmField, el.value !== undefined ? el.value : el.textContent)));
    root.querySelectorAll('[data-action="gm-add-arrival"]').forEach(btn =>
      btn.addEventListener('click', () => {
        const src = root.querySelector('#vb-gm-add-source');
        const txt = root.querySelector('#vb-gm-add-text');
        addGmArrival(src ? src.value : '', txt ? txt.value : '');
      }));
    root.querySelectorAll('#vb-gm-add-select').forEach(sel =>
      sel.addEventListener('change', () => selectGmNpc(sel.value)));
    root.querySelectorAll('[data-action="gm-save-arrival"]').forEach(btn =>
      btn.addEventListener('click', () => {
        const src = root.querySelector('#vb-gm-add-source');
        const txt = root.querySelector('#vb-gm-add-text');
        saveGmNpcEdits(src ? src.value : '', txt ? txt.value : '');
      }));
    root.querySelectorAll('[data-edit="name"]').forEach(el =>
      el.addEventListener('blur', () => {
        const b = state.buildings.find(x => x.id === el.dataset.id);
        if (b) { b.name = el.textContent.trim() || b.name; saveState(); }
      }));

    const titleInput = root.querySelector('.vb-title');
    if (titleInput) titleInput.addEventListener('change', () => {
      state.villageName = titleInput.value || DEFAULT_STATE.villageName;
      saveState();
    });

    root.querySelectorAll('[data-action="chat-post"]').forEach(btn =>
      btn.addEventListener('click', postChatMessage));
    const chatTextInput = root.querySelector('#vb-chat-text');
    if (chatTextInput) chatTextInput.addEventListener('keydown', (e) => {
      if (e.key === 'Enter') { e.preventDefault(); postChatMessage(); }
    });

    root.querySelectorAll('[data-action="note-add-text"]').forEach(btn =>
      btn.addEventListener('click', addTextNote));
    root.querySelectorAll('[data-action="note-add-image"]').forEach(btn =>
      btn.addEventListener('click', addImageNote));
    bindNotesInteractions(root);
    bindCharInteractions(root);
  }

  // Google Fonts for the display/body pairing
  const link = document.createElement('link');
  link.rel = 'stylesheet';
  link.href = 'https://fonts.googleapis.com/css2?family=Fraunces:opsz,wght@9..144,500;9..144,600&family=Spectral:wght@400;500;600&display=swap';
  document.head.appendChild(link);

  loadState();
  loadChat();
  loadNotes();
  loadCharacters();
  setInterval(refreshChat, 6000);
  setInterval(refreshNotes, 8000);
  setInterval(refreshCharacters, 8000);
})();
</script>
</body>
</html>
