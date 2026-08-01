Pipeline Builder
A visual, drag-and-drop pipeline editor (React + ReactFlow) with a FastAPI backend that validates the graph you build.
Running it
Backend
cd backend
pip install -r requirements.txt
uvicorn main:app --reload --port 8000
Frontend
cd frontend
npm install
npm run dev
Then open the printed local URL (default http://localhost:3000).
The frontend expects the backend at http://localhost:8000 by default. To point it elsewhere, add a .env file in frontend/ with:
VITE_API_BASE=https://your-backend-host
Part 1 — Node abstraction
Every node type is a plain config object handed to createNode() (src/nodes/nodeFactory.jsx), which wires it up to BaseNode (src/nodes/BaseNode.jsx) — the shared component that renders the header, fields, and handles. A brand-new node is typically ~15 lines:
export const MathNode = createNode({
  type: 'math',
  title: 'Math Op',
  accent: '#db2777',
  fields: [{ name: 'operation', label: 'Operation', type: 'select', options: [...] }],
  handles: [
    { id: 'a', type: 'target', position: Position.Left, style: { top: '35%' } },
    { id: 'b', type: 'target', position: Position.Left, style: { top: '65%' } },
    { id: 'result', type: 'source', position: Position.Right },
  ],
});
fields/handles can also be functions of (data, id), which is how the Text node grows new handles as the user types (see Part 3). Five new nodes are included to demonstrate the abstraction: Math Op, Filter, Delay, HTTP Request, and DB Query (src/nodes/*.jsx), alongside the original Input, Output, LLM, and Text nodes.
Part 2 — Styling
All nodes share one visual language defined in src/nodes/nodes.css and src/App.css: a dark silkscreen-style header, square "component lead" style ports, and a monospace label face (JetBrains Mono) paired with a system sans for field values — meant to read like a small schematic/PCB module rather than a generic card. Each node type gets a single accent color (set via a --accent-node CSS variable) that tints its header border, chip, and ports, so telling node types apart at a glance doesn't require reading labels.
Part 3 — Text node logic
src/nodes/TextNode.jsx handles both requirements:
Auto-resize: the textarea's height tracks scrollHeight on every keystroke, and the node's width grows with the longest line (clamped between a min/max) so the box is never cramped or wastefully huge.
Variable handles: input is scanned with /\{\{\s*([A-Za-z_$][A-Za-z0-9_$]*)\s*\}\}/g (a valid JS identifier inside double curly braces). Each unique match becomes a target Handle on the left edge, evenly spaced, and handles disappear again if the variable is removed from the text.
Part 4 — Backend integration
src/submit.js exports useSubmitPipeline(), a hook that reads the current nodes/edges from the store and POSTs them as JSON to /pipelines/parse, then shows an alert() with the response.
backend/main.py implements /pipelines/parse: it counts nodes and edges, and determines whether the graph is a DAG via Kahn's algorithm (repeatedly removing zero-in-degree nodes — if every node gets removed, there's no cycle). Response shape: { num_nodes, num_edges, is_dag }.
Notes
This was built and syntax-checked in a sandboxed environment without package-registry access, so npm install / pip install haven't been run end-to-end here — run them locally as the first step above.