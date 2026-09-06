# File map

| Path | Status | Why it exists |
| --- | --- | --- |
| `.altitude` | known | Connects this workshop folder to the Altitude journey. |
| `.gitignore` | known | Keeps virtual-environment and test-cache output out of Git. |
| `.python-version` | known | Pins the project to Python 3.12 for `uv`. |
| `README.md` | known | States the service's motivation, intended approach, and working product direction. |
| `.venv/` | known | Local `uv`-managed environment; rebuildable from `pyproject.toml` and `uv.lock`, never committed. |
| `data/logs/.gitkeep` | known | Keeps the future saved-log corpus directory in Git while it is empty. |
| `data/logs/sample.log` | known | Small safe fixture used to prove the environment can load a saved Klipper log. |
| `learning/plan.md` | known | Mirrors the server-planned journey and points to the current task. |
| `learning/file-map.md` | known | Records why each meaningful project file or folder exists. |
| `pyproject.toml` | known | Declares the Python package, supported interpreter, build backend, and development dependency group. |
| `src/.gitkeep` | known | Keeps the future application source directory in Git while it is empty. |
| `src/klippy_log_triage/__init__.py` | known | Makes the application package importable and documents its responsibility. |
| `src/klippy_log_triage/py.typed` | known | Marks the package as shipping type information. |
| `tests/.gitkeep` | known | Keeps the future test directory in Git while it is empty. |
| `uv.lock` | known | Pins the complete resolved dependency graph for reproducible installs. |
