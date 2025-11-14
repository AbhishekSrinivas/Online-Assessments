# System Health Check API

## What I Built

I created a Python web API that checks the health of system components organized in a Directed Acyclic Graph (DAG). The API takes a JSON input with component relationships, runs health checks asynchronously on all components, and returns an interactive HTML report with a visual graph showing which components are healthy or failed.

## How to Run It

### Step 1: Install Dependencies
Open your terminal and run:
```bash
pip install Flask networkx matplotlib
```

### Step 2: Start the Server
```bash
python Schwab-Assessment
```

You should see this message:
```
 * Running on http://127.0.0.1:5000
```

## How to Test It

Send a POST request to `http://localhost:5000/check_health` with JSON data containing your system components and their dependencies.

### Example using curl:
```bash
curl -X POST http://localhost:5000/check_health \
  -H "Content-Type: application/json" \
  -d "{\"nodes\": [\"Database\", \"API\", \"Cache\"], \"edges\": [[\"Database\", \"API\"], [\"Cache\", \"API\"]]}"
```

### What You'll Get Back

The API returns an HTML report showing:
- **Overall System Status** - Whether the system is HEALTHY or DEGRADED
- **Component Table** - Each component's health status and check duration
- **System Graph** - A visual diagram of your components with color-coded nodes (green = healthy, red = failed)

## What I Did to Meet the Requirements

✅ **Created a JSON API** - Accepts nodes and edges to build a DAG  
✅ **Validated the Graph** - Checks for cycles using NetworkX  
✅ **Used BFS Traversal** - Traverses the graph in breadth-first search order  
✅ **Async Health Checks** - All components are checked concurrently using Python's asyncio  
✅ **Displayed Results in a Table** - Shows component name, status, and duration  
✅ **Added Graph Visualization** - Visual DAG with color-coded nodes showing health status  

## Technologies Used

- **Flask** - For the web API
- **NetworkX** - For graph operations and DAG validation
- **Matplotlib** - For visualizing the system graph
- **Python asyncio** - For running health checks concurrently
- **HTML/CSS** - For the report styling
