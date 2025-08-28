# Netflix-
# Parse the uploaded notebook and generate an HTML README tailored to it.
import json, re, os, textwrap, datetime, nbformat
from pathlib import Path

nb_path = Path("/mnt/data/Netflix_project.ipynb")
project_name_guess = "Netflix Project"
description_guess = "Exploratory data analysis and insights on a Netflix dataset using Python."
imports = set()
headings = []
markdown_snippets = []

if nb_path.exists():
    nb = nbformat.read(nb_path, as_version=4)
    for cell in nb.cells:
        if cell.cell_type == "markdown":
            # Collect headings and a few early lines as a description seed
            lines = cell.source.splitlines()
            for line in lines:
                if re.match(r"^\s*#{1,6}\s+", line):
                    headings.append(re.sub(r"^\s*#{1,6}\s+", "", line).strip())
            # store first markdown block if description is not set
            if not markdown_snippets and cell.source.strip():
                markdown_snippets.append(cell.source.strip())
        elif cell.cell_type == "code":
            # Find import statements
            for match in re.finditer(r"^\s*(?:from\s+([\w\.]+)\s+import|import\s+([\w\.]+))", cell.source, flags=re.MULTILINE):
                module = match.group(1) or match.group(2)
                if module:
                    # Only keep top-level package name
                    imports.add(module.split(".")[0])

    # Prefer the first heading as project name if it looks like a title
    if headings:
        project_name_guess = headings[0]
    if markdown_snippets:
        # Use the first paragraph from the first markdown cell as description
        first_para = markdown_snippets[0].split("\n\n")[0]
        # Strip markdown formatting for description
        description_guess = re.sub(r"[#*_>`\[\]\(\)]", "", first_para).strip()

# Basic list of common data-science libs to filter noise
common_libs = ["pandas", "numpy", "matplotlib", "seaborn", "plotly", "sklearn", "scikit-learn", "statsmodels",
               "scipy", "tensorflow", "keras", "xgboost", "lightgbm", "catboost", "nltk", "spacy",
               "beautifulsoup4", "bs4", "requests", "sqlalchemy", "pyspark", "streamlit", "fastapi"]

used_libs = [lib for lib in common_libs if lib in imports]
other_libs = sorted([lib for lib in imports if lib not in used_libs])

today = datetime.date.today().strftime("%B %d, %Y")

# Build the HTML README
html = f"""<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>{project_name_guess} – README</title>
  <style>
    :root {{
      --bg: #0d1117;
      --panel: #161b22;
      --text: #c9d1d9;
      --muted: #8b949e;
      --accent: #58a6ff;
      --border: #30363d;
      --ok: #3fb950;
      --warn: #d29922;
    }}
    html, body {{ background: var(--bg); color: var(--text); font-family: Inter, system-ui, -apple-system, Segoe UI, Roboto, Arial, "Noto Sans", "Apple Color Emoji", "Segoe UI Emoji"; }}
    a {{ color: var(--accent); text-decoration: none; }}
    a:hover {{ text-decoration: underline; }}
    .container {{ max-width: 980px; margin: 2.5rem auto; padding: 0 1rem; }}
    .card {{ background: var(--panel); border: 1px solid var(--border); border-radius: 16px; padding: 24px; box-shadow: 0 0 0 1px rgba(255,255,255,0.02) inset; }}
    h1, h2, h3 {{ color: #e6edf3; margin: 0 0 12px; }}
    h1 {{ font-size: 2rem; }}
    h2 {{ font-size: 1.25rem; margin-top: 1.5rem; }}
    p, li {{ line-height: 1.7; }}
    code, pre {{ background: #0b0f14; border: 1px solid var(--border); border-radius: 10px; padding: 2px 6px; }}
    pre code {{ padding: 0; border: none; background: transparent; }}
    .badges a {{ display: inline-block; margin-right: 6px; }}
    .grid {{ display: grid; grid-template-columns: repeat(auto-fit, minmax(240px, 1fr)); gap: 16px; }}
    .kbd {{ font-family: ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, Liberation Mono, monospace; background: #111827; border: 1px solid #374151; border-bottom-width: 3px; padding: 2px 6px; border-radius: 6px; }}
    .list-compact li {{ margin: 4px 0; }}
    .muted {{ color: var(--muted); }}
    .toc a {{ display:block; padding:4px 0; }}
    .pill {{ padding:2px 8px; border:1px solid var(--border); border-radius:999px; margin-right:6px; font-size:.85rem; }}
  </style>
</head>
<body>
  <div class="container">
    <div class="card">
      <h1>{project_name_guess}</h1>
      <p class="muted">{description_guess}</p>

      <div class="badges" role="group" aria-label="project badges">
        <a href="#" aria-label="Python">
          <img alt="Python" src="https://img.shields.io/badge/python-3.x-informational.svg" />
        </a>
        <a href="#" aria-label="License">
          <img alt="License: MIT" src="https://img.shields.io/badge/License-MIT-green.svg" />
        </a>
        <a href="#" aria-label="Last updated">
          <img alt="Updated" src="https://img.shields.io/badge/updated-{today.replace(' ', '%20')}-blue.svg" />
        </a>
      </div>

      <h2>📌 Overview</h2>
      <p>
        This repository contains a Jupyter Notebook (<code>Netflix_project.ipynb</code>) focused on data exploration,
        visualization, and insights related to Netflix content. The analysis includes data cleaning, descriptive statistics,
        and charting to uncover trends (e.g., content types, genres, release years, and ratings).
      </p>

      <div class="grid">
        <div>
          <h2>🗂 Contents</h2>
          <ul class="list-compact">
            <li><code>Netflix_project.ipynb</code> – main analysis notebook.</li>
            <li><code>data/</code> – (optional) place raw/processed datasets here.</li>
            <li><code>images/</code> – (optional) exported figures.</li>
            <li><code>requirements.txt</code> – Python dependencies.</li>
          </ul>
        </div>
        <div>
          <h2>🔗 Quick Links</h2>
          <div class="toc">
            <a href="#getting-started">Getting Started</a>
            <a href="#dataset">Dataset</a>
            <a href="#usage">Usage</a>
            <a href="#results">Results & Visuals</a>
            <a href="#project-structure">Project Structure</a>
            <a href="#contributing">Contributing</a>
            <a href="#license">License</a>
          </div>
        </div>
      </div>

      <h2 id="getting-started">⚙️ Getting Started</h2>
      <p>Clone the repository and set up a virtual environment:</p>
<pre><code>git clone &lt;your-repo-url&gt;
cd &lt;your-repo&gt;

# create and activate venv (Windows)
python -m venv .venv
.venv\\Scripts\\activate

# macOS/Linux
python3 -m venv .venv
source .venv/bin/activate
</code></pre>

      <h2>📦 Install Requirements</h2>
      <p>Create a <code>requirements.txt</code> using the libraries detected from the notebook, or adjust as needed:</p>
<pre><code>{os.linesep.join(sorted(set(used_libs + other_libs)) or ["pandas","numpy","matplotlib","seaborn"])}
</code></pre>

      <h2 id="dataset">🧺 Dataset</h2>
      <p>
        Add your dataset under <code>data/</code> and update the notebook path accordingly. If the dataset is public,
        include a link and brief description here (columns, row count, and any important caveats).
      </p>

      <h2 id="usage">▶️ Usage</h2>
      <p>Open the notebook and run cells top-to-bottom:</p>
<pre><code>jupyter lab
# or
jupyter notebook
</code></pre>
      <p>If you prefer a script workflow, you can export the notebook as a Python script:</p>
<pre><code>jupyter nbconvert --to script Netflix_project.ipynb
python Netflix_project.py
</code></pre>

      <h2 id="results">📊 Results & Visuals</h2>
      <p>
        Include key plots or tables here (e.g., distribution of content by year, top genres, rating breakdown).
        Save images into <code>images/</code> and embed them, for example:
      </p>
      <p>
        &lt;img src="images/top-genres.png" alt="Top genres" width="600" /&gt;
      </p>

      <h2 id="project-structure">🏗 Project Structure</h2>
<pre><code>.
├─ data/
├─ images/
├─ Netflix_project.ipynb
├─ requirements.txt
└─ README.md (this file as HTML)
</code></pre>

      <h2>🧪 Reproducibility Tips</h2>
      <ul>
        <li>Record your package versions: <span class="kbd">pip freeze &gt; requirements.txt</span></li>
        <li>Set a random seed where applicable for consistent results.</li>
        <li>Document data cleaning steps clearly.</li>
      </ul>

      <h2 id="contributing">🤝 Contributing</h2>
      <p>Contributions, issues, and feature requests are welcome! Feel free to open an
      <a href="https://github.com/issues">issue</a> or submit a PR.</p>

      <h2 id="license">📄 License</h2>
      <p>Distributed under the MIT License. See <code>LICENSE</code> for more information.</p>

      <hr />
      <p class="muted">Generated on {today}. Replace placeholders (repo URL, dataset link, images) as needed.</p>
    </div>
  </div>
</body>
</html>
"""

out_path = Path("/mnt/data/README.html")
out_path.write_text(html, encoding="utf-8")
out_path.as_posix()
<!doctype html>


