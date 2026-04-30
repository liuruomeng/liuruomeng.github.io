---
layout: default
title: "CV"
permalink: /cv/
description: "Curriculum Vitae of Ruomeng Liu."
---

{% assign cv_url = '/assets/pdfs/cv_onepage.pdf' | relative_url %}

<p style="color: hsla(0, 0%, 0%, 0.65); font-size: 0.95em;">This is an abbreviated one-page CV. A full version is available upon request.</p>

<div class="pdf-viewer" data-pdf-src="{{ cv_url }}">
  <div class="pdf-toolbar" role="toolbar" aria-label="PDF controls">
    <div class="pdf-toolbar-group">
      <button type="button" class="pdf-btn" data-action="zoom-out" aria-label="Zoom out">&minus;</button>
      <span class="pdf-zoom-label" aria-live="polite">100%</span>
      <button type="button" class="pdf-btn" data-action="zoom-in" aria-label="Zoom in">+</button>
      <button type="button" class="pdf-btn" data-action="fit-width">Fit width</button>
    </div>
    <div class="pdf-toolbar-group">
      <span class="pdf-page-info"><span class="pdf-current-page">–</span> / <span class="pdf-total-pages">–</span></span>
      <a class="pdf-btn" href="{{ cv_url }}" target="_blank" rel="noopener">Open</a>
      <a class="pdf-btn" href="{{ cv_url }}" download>Download</a>
    </div>
  </div>

  <div class="pdf-pages" aria-busy="true">
    <p class="pdf-loading">Loading PDF…</p>
  </div>

  <noscript>
    <p>JavaScript is disabled. <a href="{{ cv_url }}">Open the CV (PDF)</a>.</p>
  </noscript>
</div>

<script type="module">
  import * as pdfjsLib from 'https://cdn.jsdelivr.net/npm/pdfjs-dist@4.7.76/build/pdf.min.mjs';
  pdfjsLib.GlobalWorkerOptions.workerSrc =
    'https://cdn.jsdelivr.net/npm/pdfjs-dist@4.7.76/build/pdf.worker.min.mjs';

  const viewer = document.querySelector('.pdf-viewer');
  const pagesEl = viewer.querySelector('.pdf-pages');
  const zoomLabel = viewer.querySelector('.pdf-zoom-label');
  const totalEl = viewer.querySelector('.pdf-total-pages');
  const currentEl = viewer.querySelector('.pdf-current-page');
  const src = viewer.dataset.pdfSrc;

  let pdfDoc = null;
  let scale = 1.25;
  let renderToken = 0;
  const minScale = 0.5;
  const maxScale = 3.0;
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting && entry.intersectionRatio > 0.4) {
        currentEl.textContent = entry.target.dataset.pageNumber;
      }
    });
  }, { threshold: [0.4, 0.6] });

  async function renderAll() {
    const myToken = ++renderToken;
    observer.disconnect();
    pagesEl.innerHTML = '';
    pagesEl.setAttribute('aria-busy', 'true');
    const dpr = window.devicePixelRatio || 1;
    for (let n = 1; n <= pdfDoc.numPages; n++) {
      if (myToken !== renderToken) return;
      const page = await pdfDoc.getPage(n);
      const viewport = page.getViewport({ scale });
      const canvas = document.createElement('canvas');
      canvas.className = 'pdf-page';
      canvas.dataset.pageNumber = n;
      canvas.width = Math.floor(viewport.width * dpr);
      canvas.height = Math.floor(viewport.height * dpr);
      canvas.style.width = viewport.width + 'px';
      canvas.style.height = viewport.height + 'px';
      pagesEl.appendChild(canvas);
      await page.render({
        canvasContext: canvas.getContext('2d'),
        viewport,
        transform: dpr !== 1 ? [dpr, 0, 0, dpr, 0, 0] : null,
      }).promise;
      observer.observe(canvas);
    }
    pagesEl.setAttribute('aria-busy', 'false');
    zoomLabel.textContent = Math.round(scale * 80) + '%';
  }

  async function fitWidth() {
    if (!pdfDoc) return;
    const page = await pdfDoc.getPage(1);
    const baseViewport = page.getViewport({ scale: 1 });
    const containerWidth = pagesEl.clientWidth - 16;
    scale = Math.max(minScale, Math.min(maxScale, containerWidth / baseViewport.width));
    return renderAll();
  }

  viewer.querySelector('[data-action="zoom-in"]').addEventListener('click', () => {
    scale = Math.min(maxScale, scale + 0.2);
    renderAll();
  });
  viewer.querySelector('[data-action="zoom-out"]').addEventListener('click', () => {
    scale = Math.max(minScale, scale - 0.2);
    renderAll();
  });
  viewer.querySelector('[data-action="fit-width"]').addEventListener('click', fitWidth);

  try {
    pdfDoc = await pdfjsLib.getDocument(src).promise;
    totalEl.textContent = pdfDoc.numPages;
    currentEl.textContent = '1';
    await fitWidth();
  } catch (err) {
    pagesEl.innerHTML = '<p>Could not load PDF. <a href="' + src + '">Open it directly</a>.</p>';
    console.error(err);
  }
</script>
