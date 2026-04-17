Show a summary of the current Ghost-Bridge setup in this project.

1. Check if `.ghost-bridge/` exists. If not, say "No bridges generated yet."

2. If it exists, list every file inside `.ghost-bridge/bridges/` and show:
   - Bridge name
   - Source language (inferred from file extension)
   - Target language (inferred from file extension)
   - File size

3. Check `.ghost-bridge/.venv/` — if present, show Python version inside it:
   `.ghost-bridge/.venv/bin/python --version`

4. Show which bridge pairs are available in this repo's `bridges/` directory
   (read the directory listing of `bridges/`).

5. Print a quick-start reminder:
   "To generate a new bridge: /ghost-bridge <source> <target> <name> '<capability>' '<deps>'"
   Example: /ghost-bridge typescript python image_processor 'resize images' 'Pillow'
