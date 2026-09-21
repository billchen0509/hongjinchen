---
layout: page
permalink: /publications/
title: Publications
description: publications by categories in reversed chronological order.
nav: true
nav_order: 2
chart:
  chartjs: true
---

<!-- _pages/publications.md -->

{% assign scholar = site.data.citations %}
{% if scholar and scholar.metadata %}
{% assign scholar_metrics = scholar.metadata %}

<section id="scholar-metrics" aria-label="Google Scholar citation metrics">
  <style>
    #scholar-metrics .sc-metrics {
      display: grid;
      grid-template-columns: repeat(3, minmax(0, 1fr));
      gap: 16px;
    }
    @media (max-width: 640px) {
      #scholar-metrics .sc-metrics {
        grid-template-columns: 1fr;
      }
    }
    #scholar-metrics .sc-card {
      border: 1px solid rgba(128, 128, 128, 0.4);
      border-radius: 8px;
      padding: 16px;
    }
    #scholar-metrics .sc-label {
      margin: 0 0 4px;
      font-size: 0.8rem;
      color: #6b7280;
    }
    #scholar-metrics .sc-value {
      margin: 0;
      font-size: 2rem;
      font-weight: 600;
      line-height: 1.1;
    }
    #scholar-metrics .sc-chart-box {
      margin-top: 16px;
      border: 1px solid rgba(128, 128, 128, 0.4);
      border-radius: 8px;
      padding: 16px;
    }
    #scholar-metrics .sc-footer {
      margin-top: 8px;
      font-size: 0.8rem;
      color: #6b7280;
    }
    #scholar-metrics .sc-footer a {
      color: inherit;
      text-decoration: underline;
    }
  </style>

  <div class="sc-metrics">
    <div class="sc-card">
      <p class="sc-label">Citations in total</p>
      <p class="sc-value">{{ scholar_metrics.citedby | default: 0 }}</p>
    </div>
    <div class="sc-card">
      <p class="sc-label">h-index</p>
      <p class="sc-value">{{ scholar_metrics.hindex | default: 0 }}</p>
    </div>
    <div class="sc-card">
      <p class="sc-label">i10-index</p>
      <p class="sc-value">{{ scholar_metrics.i10index | default: 0 }}</p>
    </div>
  </div>

  {% if scholar_metrics.cites_per_year %}
  {% assign scholar_cp = scholar_metrics.cites_per_year | sort %}
  {% if scholar_cp.size > 0 %}
  <div class="sc-chart-box">
    <div style="position: relative; height: 340px;">
      <canvas id="scholar-citations-chart" role="img" aria-label="Google Scholar citations per year"></canvas>
    </div>
    <p id="scholar-chart-fallback" style="display: none; margin-top: 8px; font-size: 0.8rem; color: #6b7280;">
      Chart could not be loaded. Please check your network connection.
    </p>
  </div>

  <script>
    (function () {
      var initialized = false;
      var labels = [{% for cp in scholar_cp %}{{ cp[0] | jsonify }}{% unless forloop.last %},{% endunless %}{% endfor %}];
      var values = [{% for cp in scholar_cp %}{{ cp[1] | jsonify }}{% unless forloop.last %},{% endunless %}{% endfor %}];

      function render() {
        if (initialized) return;
        var canvas = document.getElementById('scholar-citations-chart');
        var fallback = document.getElementById('scholar-chart-fallback');
        if (!canvas) return;
        if (!window.Chart) {
          if (fallback) fallback.style.display = 'block';
          return;
        }
        if (fallback) fallback.style.display = 'none';
        initialized = true;

        var cumulative = [];
        var running = 0;
        values.forEach(function (value) {
          running += value;
          cumulative.push(running);
        });

        Chart.defaults.color = '#6b7280';

        new Chart(canvas, {
          data: {
            labels: labels,
            datasets: [
              {
                type: 'bar',
                label: 'Citations per year',
                data: values,
                backgroundColor: 'rgba(59, 130, 246, 0.45)',
                borderColor: 'rgba(59, 130, 246, 1)',
                borderWidth: 1,
                yAxisID: 'y',
              },
              {
                type: 'line',
                label: 'Cumulative citations',
                data: cumulative,
                borderColor: 'rgba(16, 185, 129, 1)',
                backgroundColor: 'rgba(16, 185, 129, 0.12)',
                fill: true,
                tension: 0.25,
                pointRadius: 3,
                pointBackgroundColor: 'rgba(16, 185, 129, 1)',
                yAxisID: 'y1',
              },
            ],
          },
          options: {
            responsive: true,
            maintainAspectRatio: false,
            interaction: { mode: 'index', intersect: false },
            plugins: { legend: { position: 'bottom' } },
            scales: {
              x: {
                grid: { color: 'rgba(128, 128, 128, 0.25)' },
                title: { display: true, text: 'Year' },
              },
              y: {
                beginAtZero: true,
                position: 'left',
                grid: { color: 'rgba(128, 128, 128, 0.25)' },
                title: { display: true, text: 'Citations per year' },
              },
              y1: {
                beginAtZero: true,
                position: 'right',
                grid: { drawOnChartArea: false },
                title: { display: true, text: 'Total citations' },
              },
            },
          },
        });
      }

      if (document.readyState === 'loading') {
        document.addEventListener('DOMContentLoaded', render);
      } else {
        render();
      }

      // Chart.js is injected by the al_charts plugin. If it arrives after this
      // inline script, poll briefly instead of leaving an empty canvas.
      if (!window.Chart) {
        var attempts = 0;
        var timer = setInterval(function () {
          attempts += 1;
          if (window.Chart || attempts >= 40) {
            clearInterval(timer);
            render();
          }
        }, 100);
      }
    })();
  </script>
  {% endif %}
  {% endif %}

  <p class="sc-footer">
    Last updated: {{ scholar_metrics.last_updated }} ·
    <a href="https://scholar.google.com/citations?user={{ site.data.socials.scholar_userid }}" target="_blank" rel="noopener noreferrer">
      Google Scholar profile
    </a>
  </p>
</section>

{% endif %}

<!-- Bibsearch Feature -->

{% include bib_search.liquid %}

<div class="publications">

{% bibliography %}

</div>
