# Overarch CLI Usage Reference

This document provides comprehensive details on using the Overarch command-line interface.

## Basic Command Structure

```bash
java -jar overarch.jar [options]
```

Or if installed via Homebrew:

```bash
overarch [options]
```

## Core Options

### Model Directory

**Option:** `-m, --model-dir PATH`

**Description:** Specifies the directory containing model EDN files. Overarch recursively loads all `.edn` files from this directory.

**Example:**
```bash
overarch -m vault/architecture/
overarch -m models/
overarch --model-dir /path/to/models
```

**Default:** Current directory if not specified.

**Best Practice:** Organize models in a dedicated directory structure.

## Rendering Options

### Render Format

**Option:** `-r, --render-format FORMAT`

**Description:** Specifies output format for rendering diagrams.

**Values:**
- `all` - Generate all formats
- `plantuml` - PlantUML diagrams (.puml)
- `graphviz` - GraphViz diagrams (.dot)
- `markdown` - Markdown documentation (.md)

**Examples:**
```bash
overarch -m models/ -r all
overarch -m models/ -r plantuml
overarch -m models/ -r graphviz
overarch -m models/ -r markdown
```

**Multiple formats:**
```bash
overarch -m models/ -r plantuml -r markdown
```

### Render Directory

**Option:** `-R, --render-dir DIRNAME`

**Description:** Specifies output directory for rendered files.

**Example:**
```bash
overarch -m models/ -r all -R output/diagrams/
overarch -m models/ -r plantuml -R exports/plantuml/
```

**Default:** Creates output in subdirectories of current directory:
- `plantuml/` for PlantUML files
- `graphviz/` for GraphViz files
- `markdown/` for Markdown files

## Export Options

### Export Format

**Option:** `-x, --export-format FORMAT`

**Description:** Exports model to alternative data formats.

**Values:**
- `json` - Export to JSON format
- `structurizr` - Export to Structurizr workspace format (experimental)

**Examples:**
```bash
overarch -m models/ -x json
overarch -m models/ -x structurizr
overarch -m models/ -x json -x structurizr
```

### Export Directory

**Option:** `-X, --export-dir DIRNAME`

**Description:** Specifies output directory for exports.

**Example:**
```bash
overarch -m models/ -x json -X exports/json/
```

## Query and Selection

### Select Elements

**Option:** `-s, --select-elements CRITERIA`

**Description:** Queries and prints model elements matching criteria. Uses same selection syntax as view specifications.

**Examples:**

**Select by element type:**
```bash
overarch -m models/ -s '{:el :system}'
```

**Select by namespace:**
```bash
overarch -m models/ -s '{:namespace :banking.internet-banking}'
```

**Select by technology:**
```bash
overarch -m models/ -s '{:tech "Java"}'
```

**Select by tags:**
```bash
overarch -m models/ -s '{:tags #{:frontend}}'
```

**Complex selection:**
```bash
overarch -m models/ -s '{:el :container :tech "PostgreSQL" :!external? true}'
```

**Output:** Prints matching elements as EDN to stdout.

## Template-Based Generation

### Generation Configuration

**Option:** `-g, --generation-config FILE`

**Description:** Specifies template generation configuration file.

**Example:**
```bash
overarch -m models/ -g generation-config.edn
```

**Configuration File Format:**

```clojure
[{:selection {:el :system}           ; Element selection criteria
  :template "templates/system.cmb"   ; Template file path
  :engine :combsci                   ; Template engine (:comb or :combsci)
  :per-element true                  ; Generate per element
  :subdir "generated"                ; Output subdirectory
  :extension "md"}                   ; File extension

 {:selection {:el :container}
  :template "templates/api-doc.cmb"
  :engine :combsci
  :per-namespace true
  :subdir "docs/apis"
  :extension "md"}]
```

### Template Directory

**Option:** `-T, --template-dir DIRNAME`

**Description:** Specifies directory containing template files.

**Example:**
```bash
overarch -m models/ -g config.edn -T templates/
```

### Generation Directory

**Option:** `-G, --generation-dir DIRNAME`

**Description:** Specifies base output directory for generated artifacts.

**Example:**
```bash
overarch -m models/ -g config.edn -G generated/
```

## Watch Mode

### Watch for Changes

**Option:** `-w, --watch`

**Description:** Watches model files for changes and automatically regenerates output.

**Example:**
```bash
overarch -m models/ -r all --watch
overarch -m models/ -r plantuml -w
```

**Behavior:**
- Monitors all `.edn` files in model directory
- Regenerates on any file change
- Runs continuously until interrupted (Ctrl+C)

**Use Cases:**
- Development workflow
- Live diagram updates
- Iterative modeling

**Best Practice:** Run in dedicated terminal while editing models in your IDE.

## Debug and Logging

### Debug Output

**Option:** `--debug`

**Description:** Enables verbose debug output.

**Example:**
```bash
overarch -m models/ -r all --debug
```

**Output:** Detailed logging of:
- File loading
- Model parsing
- Element resolution
- View processing
- Rendering steps

### Quiet Mode

**Option:** `-q, --quiet`

**Description:** Suppresses non-error output.

**Example:**
```bash
overarch -m models/ -r all --quiet
```

## Version and Help

### Version

**Option:** `--version`

**Description:** Prints Overarch version.

**Example:**
```bash
overarch --version
```

### Help

**Option:** `-h, --help`

**Description:** Prints command-line help.

**Example:**
```bash
overarch --help
```

## Common Command Patterns

### Initial Model Rendering

Generate all diagram formats from models:

```bash
overarch -m vault/architecture/ -r all
```

### Development Workflow

Watch mode with specific format:

```bash
overarch -m models/ -r plantuml --watch
```

### Production Export

Export to multiple formats with organized output:

```bash
overarch -m models/ \
  -r plantuml -R output/diagrams/plantuml/ \
  -r markdown -R output/docs/ \
  -x json -X output/exports/
```

### Query Model

Find all systems using Java:

```bash
overarch -m models/ -s '{:el :system :tech "Java"}'
```

Find internal containers:

```bash
overarch -m models/ -s '{:el :container :!external? true}'
```

### Generate Documentation

Using templates:

```bash
overarch -m models/ \
  -g generation-config.edn \
  -T templates/ \
  -G generated/docs/
```

### Validate Model

Check model loads without rendering:

```bash
overarch -m models/ --debug
```

If models load successfully, you'll see "Model loaded" in debug output.

## PlantUML Integration

After generating PlantUML files, convert to images:

### Single File

```bash
plantuml diagrams/context-view.puml
```

### Batch Conversion

```bash
plantuml diagrams/*.puml
```

### Specific Format

```bash
plantuml -tpng diagrams/*.puml    # PNG
plantuml -tsvg diagrams/*.puml    # SVG
plantuml -tpdf diagrams/*.puml    # PDF
```

### Watch Mode (PlantUML)

```bash
plantuml -tpng -checkonly -realtimediagram diagrams/
```

**Note:** For live updates, run both Overarch and PlantUML in watch mode.

## GraphViz Integration

After generating GraphViz files, convert to images:

### Single File

```bash
dot -Tpng diagrams/concept-map.dot -o diagrams/concept-map.png
```

### Batch Conversion

```bash
for file in diagrams/*.dot; do
  dot -Tpng "$file" -o "${file%.dot}.png"
done
```

### Different Layouts

```bash
neato -Tpng diagrams/concept-map.dot -o output.png
fdp -Tpng diagrams/concept-map.dot -o output.png
circo -Tpng diagrams/concept-map.dot -o output.png
```

## Workflow Examples

### First-Time Setup

```bash
# 1. Create model directory
mkdir -p vault/architecture/models

# 2. Create initial model
# (Edit vault/architecture/models/model.edn)

# 3. Generate all outputs
overarch -m vault/architecture/models/ -r all

# 4. Convert PlantUML to PNG
plantuml vault/architecture/plantuml/*.puml
```

### Iterative Development

```bash
# Terminal 1: Watch and regenerate diagrams
overarch -m models/ -r plantuml --watch

# Terminal 2: Watch and convert to PNG
plantuml -tpng -checkonly -realtimediagram plantuml/

# Terminal 3: Your text editor
# Edit models/*.edn files
```

### CI/CD Integration

```bash
#!/bin/bash
# Generate and validate architecture diagrams

set -e

echo "Generating architecture diagrams..."
overarch -m models/ -r all -R output/diagrams/

echo "Converting PlantUML to PNG..."
plantuml -tpng output/diagrams/plantuml/*.puml

echo "Exporting model to JSON..."
overarch -m models/ -x json -X output/exports/

echo "Architecture artifacts generated successfully!"
```

### Documentation Generation

```bash
# Generate comprehensive documentation
overarch -m models/ \
  -r markdown -R docs/architecture/ \
  -r plantuml -R docs/diagrams/source/ \
  -x json -X docs/api/

# Convert diagrams
plantuml -tsvg docs/diagrams/source/*.puml -o ../rendered/

# Generate from templates
overarch -m models/ \
  -g templates/documentation-config.edn \
  -T templates/ \
  -G docs/generated/
```

## Configuration Best Practices

### Project Structure

```
project/
├── models/                    # Model EDN files
│   ├── actors.edn
│   ├── systems.edn
│   ├── containers.edn
│   ├── relationships.edn
│   └── views.edn
├── templates/                 # Generation templates
│   ├── system-doc.cmb
│   └── api-spec.cmb
├── generation-config.edn      # Generation configuration
├── output/                    # Generated outputs
│   ├── diagrams/
│   │   ├── plantuml/
│   │   ├── graphviz/
│   │   └── markdown/
│   └── exports/
│       └── json/
└── scripts/
    ├── generate.sh           # Generation script
    └── watch.sh              # Development watch script
```

### Makefile Example

```makefile
.PHONY: all clean diagrams watch

MODEL_DIR = models
OUTPUT_DIR = output

all: diagrams export

diagrams:
	overarch -m $(MODEL_DIR) -r all -R $(OUTPUT_DIR)/diagrams/
	plantuml -tpng $(OUTPUT_DIR)/diagrams/plantuml/*.puml

export:
	overarch -m $(MODEL_DIR) -x json -X $(OUTPUT_DIR)/exports/

watch:
	overarch -m $(MODEL_DIR) -r plantuml --watch

clean:
	rm -rf $(OUTPUT_DIR)
```

## Troubleshooting

### Model Not Loading

**Issue:** "No model files found"

**Solutions:**
- Check `-m` path is correct
- Ensure `.edn` file extension
- Verify file permissions
- Use `--debug` to see loading details

### Invalid EDN Syntax

**Issue:** "Failed to parse model"

**Solutions:**
- Validate EDN syntax (balanced brackets, keywords, strings)
- Use Clojure-aware editor with syntax checking
- Test with: `clojure -M -e "(require '[clojure.edn :as edn]) (edn/read-string (slurp \"model.edn\"))"`

### View Not Rendering

**Issue:** "No elements in view"

**Solutions:**
- Check selection criteria match elements
- Verify element IDs exist
- Use empty selection `{}` to see all elements
- Use `-s` to query what elements exist

### Missing Relationships

**Issue:** Relationships not appearing

**Solutions:**
- Verify `:from` and `:to` IDs exist
- Ensure view includes both source and target
- Add `:include [:relations]` to view spec
- Check relationship is correct type for view

## Performance Tips

### Large Models

For models with 100+ elements:

1. **Split by namespace:** Organize into multiple files
2. **Use focused views:** Select specific namespaces or tags
3. **Parallel processing:** Render different formats in parallel
4. **Incremental updates:** Use watch mode instead of full regeneration

### CI/CD Optimization

```bash
# Only regenerate if model changed
if git diff --quiet HEAD models/; then
  echo "No model changes, skipping regeneration"
else
  echo "Model changed, regenerating..."
  overarch -m models/ -r all
fi
```

## Environment Variables

While Overarch doesn't use environment variables directly, you can use them in scripts:

```bash
#!/bin/bash
MODEL_DIR=${MODEL_DIR:-models}
OUTPUT_DIR=${OUTPUT_DIR:-output}
RENDER_FORMAT=${RENDER_FORMAT:-all}

overarch -m "$MODEL_DIR" -r "$RENDER_FORMAT" -R "$OUTPUT_DIR"
```

## Advanced Usage

### Programmatic Access

Use Overarch as a library in Clojure projects:

```clojure
(require '[org.soulspace.overarch.core :as overarch])

(def model (overarch/load-model "models/"))
(overarch/render-views model {:format :plantuml :output-dir "output/"})
```

### Custom Validation

Query model to validate architectural rules:

```bash
# Find containers without technology specified
overarch -m models/ -s '{:el :container}' | \
  clojure -M -e "(filter #(empty? (:tech %)) (edn/read-string (slurp *in*)))"
```

### Integration with Other Tools

**Structurizr:**
```bash
# Export to Structurizr format
overarch -m models/ -x structurizr -X structurizr/

# Upload to Structurizr (requires Structurizr CLI)
structurizr-cli push -w structurizr/workspace.json
```

**Confluence:**
```bash
# Generate markdown, convert to Confluence format
overarch -m models/ -r markdown -R confluence/
# Use Confluence API to upload
```

**Git Hooks:**
```bash
# Pre-commit hook to validate model
#!/bin/bash
overarch -m models/ --quiet || {
  echo "Architecture model validation failed"
  exit 1
}
```
