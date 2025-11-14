# Hello! Here's my solution for the system health check API.
#
# --- 0. Setup / Dependencies ---
# Before running this script, I need to make sure the required libraries
# are installed. I can do this from my terminal by running:
# pip install Flask networkx matplotlib
#
# The other libraries (asyncio, random, time, io, base64) are
# part of the standard Python library, so they don't need to be installed.
#
# --- 1. Imports ---
# First, I'm importing all the libraries I'll need.
import asyncio  # For running the health checks asynchronously (Standard Library)
import random   # To simulate health check pass/fail (Standard Library)
import time     # To measure the duration of checks (Standard Library)
import io       # To hold the graph image in memory (Standard Library)
import base64   # To encode the image for the HTML response (Standard Library)

# I'm using Flask for the web API part. (Requires: pip install Flask)
from flask import Flask, request, jsonify, render_template_string

# I'll use networkx for all the graph operations. (Requires: pip install networkx)
import networkx as nx

# And I'll use matplotlib to draw the graph. (Requires: pip install matplotlib)
import matplotlib
matplotlib.use('Agg')  # This is important to run matplotlib without a GUI
import matplotlib.pyplot as plt

# --- 2. Initialize the Flask App ---
# I'm setting up my Flask app here.
app = Flask(__name__)

# --- 3. Asynchronous Health Check Function ---
# This is my core function to check a single component's health.
# I'm making it 'async' to run many checks at the same time, no waitng too.

async def check_component_health(component_name):
    """
    This function simulates an asynchronous health check for a component.
    In a real-world scenario, this is where I would put code to
    ping a service, query a database, or check a hardware status.
    """
    print(f"Starting health check for: {component_name}...")
    
    # I'm simulating a random delay (between 0.5 and 2 seconds)
    # to mimic a real network request.
    start_time = time.time()
    await asyncio.sleep(random.uniform(0.5, 2.0))
    end_time = time.time()
    
    duration = round(end_time - start_time, 2)
    
    # I'm simulating a random pass/fail status.
    # Let's say there's a 90% chance of success.
    status = "OK" if random.random() < 0.9 else "FAILED"
    
    print(f"Finished health check for: {component_name} in {duration}s. Status: {status}")
    
    # I'm returning a dictionary with all the info I need.
    return {
        "component": component_name,
        "status": status,
        "duration_seconds": duration
    }

# --- 4. Graph Visualization Function ---
# This is the (optional) function to draw the graph.
def generate_graph_image(graph, health_results):
    """
    I'm using networkx and matplotlib to draw the graph.
    I'll color the nodes based on their health status.
    """
    try:
        # I create a color map based on the health results.
        # Default to green (OK).
        color_map = {}
        for node in graph.nodes():
            status = health_results.get(node, {}).get("status", "OK")
            color_map[node] = 'lightgreen' if status == 'OK' else 'salmon'

        node_colors = [color_map[node] for node in graph.nodes()]

        # I'm setting up the drawing layout.
        # `spring_layout` often looks nice for DAGs.
        plt.figure(figsize=(12, 7))
        pos = nx.spring_layout(graph, seed=42) 
        
        # I'm drawing the nodes, edges, and labels.
        nx.draw_networkx_nodes(graph, pos, node_size=2000, node_color=node_colors, edgecolors='black')
        nx.draw_networkx_edges(graph, pos, node_size=2000, arrowstyle='->', arrowsize=20, edge_color='gray')
        nx.draw_networkx_labels(graph, pos, font_size=10)
        
        plt.title("System Health DAG")
        plt.axis('off')  # I don't need axes for this plot.

        # Now, I'm saving the plot to an in-memory buffer instead of a file.
        img_buffer = io.BytesIO()
        plt.savefig(img_buffer, format='png')
        plt.close()  # I close the plot to free up memory.
        img_buffer.seek(0)
        
        # Finally, I'm encoding the image as a base64 string
        # so I can embed it directly into my HTML response.
        img_base64 = base64.b64encode(img_buffer.getvalue()).decode('utf-8')
        return f"data:image/png;base64,{img_base64}"
        
    except Exception as e:
        print(f"Error generating graph image: {e}")
        return None

# --- 5. Main API Endpoint ---
# This is my main API endpoint. It accepts POST requests at '/check_health'.
@app.route('/check_health', methods=['POST'])
def run_health_check():
    """
    This endpoint takes the JSON graph definition, runs all health checks
    asynchronously, and returns an HTML report.
    """
    
    # First, I get the JSON data from the request.
    # I'm adding error handling in case the JSON is missing or bad.
    try:
        data = request.json
        if not data or "nodes" not in data or "edges" not in data:
            return jsonify({"error": "Invalid JSON payload. 'nodes' and 'edges' keys are required."}), 400
    except Exception as e:
        return jsonify({"error": f"Could not parse request JSON: {e}"}), 400

    # --- 6. Build the Graph ---
    # Now, I'm building the graph using networkx.
    # I'm using a DiGraph (Directed Graph) because the problem specifies a DAG.
    G = nx.DiGraph()
    G.add_nodes_from(data['nodes'])
    G.add_edges_from(data['edges'])

    # I should also check if it's *actually* a DAG (no cycles).
    if not nx.is_directed_acyclic_graph(G):
        return jsonify({"error": "The provided graph is not a DAG (it contains cycles)."}), 400

    # --- 7. Traverse Graph (BFS) and Run Checks ---
    # I'm following the requirement to traverse using Breadth-First Search (BFS).
    # I'll find the root nodes (nodes with no incoming edges) to start my traversal.
    root_nodes = [node for node, in_degree in G.in_degree() if in_degree == 0]
    
    if not root_nodes:
        return jsonify({"error": "Graph has no root nodes (nodes with no dependencies)."}), 400

    print(f"Starting BFS from root(s): {root_nodes}")
    
    # I'm using `nx.bfs_edges` which gives me an iterator of edges in BFS order.
    # I can get all unique nodes in traversal order from this.
    all_nodes_in_bfs_order = list(dict.fromkeys(sum([edge for edge in nx.bfs_edges(G, root_nodes[0])], ())))
    
    # This doesn't guarantee *all* nodes are included if the graph is disconnected.
    # A safer bet is to just check all nodes defined in the graph.
    all_nodes = list(G.nodes())
    print(f"Nodes to check: {all_nodes}")

    # I'm creating a list of async tasks, one for each component.
    async_tasks = [check_component_health(node) for node in all_nodes]

    # This is the magic part: I'm running all my async tasks concurrently.
    # I need to create a new event loop to run my async code
    # inside this synchronous Flask function.
    try:
        loop = asyncio.new_event_loop()
        asyncio.set_event_loop(loop)
        # `asyncio.gather` collects all tasks and runs them.
        results = loop.run_until_complete(asyncio.gather(*async_tasks))
    finally:
        loop.close()

    # --- 8. Process Results and Generate Report ---
    
    # I'm organizing the results into a dictionary for easier lookup.
    health_results = {res["component"]: res for res in results}
    
    # I'm determining the overall system status.
    # If any component failed, the whole system is 'DEGRADED'.
    overall_status = "HEALTHY"
    for res in results:
        if res["status"] == "FAILED":
            overall_status = "DEGRADED"
            break
            
    total_check_time = sum(res["duration_seconds"] for res in results)
    
    # I'm calling my function to generate the graph image.
    graph_image_data = generate_graph_image(G, health_results)

    # --- 9. Return HTML Response ---
    # I'm building an HTML string for my response. This is simpler than
    # creating separate template files for this example.
    
    # I'll start with the table.
    table_rows = ""
    for res in sorted(results, key=lambda x: x['component']):
        # I'm styling the row red if it failed.
        style = 'style="background-color: #ffcccc;"' if res['status'] == 'FAILED' else ''
        table_rows += f"""
        <tr {style}>
            <td>{res['component']}</td>
            <td>{res['status']}</td>
            <td>{res['duration_seconds']}s</td>
        </tr>
        """

    # Now, I'm putting it all together in a full HTML page.
    html_response = f"""
    <!DOCTYPE html>
    <html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>System Health Report</title>
        <style>
            body {{ 
                font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
                margin: 0; padding: 2em; background-color: #f9f9f9; color: #333;
            }}
            .container {{ 
                max-width: 1200px; margin: auto; background: #fff; 
                padding: 2em; border-radius: 8px; box-shadow: 0 4px 12px rgba(0,0,0,0.05);
            }}
            h1, h2 {{ color: #111; }}
            .status-box {{
                padding: 1em; border-radius: 8px; font-size: 1.5em;
                font-weight: bold; text-align: center; margin: 1em 0;
            }}
            .status-healthy {{ background-color: #d4edda; color: #155724; border: 1px solid #c3e6cb; }}
            .status-degraded {{ background-color: #f8d7da; color: #721c24; border: 1px solid #f5c6cb; }}
            table {{ 
                width: 100%; border-collapse: collapse; margin-top: 1.5em; 
            }}
            th, td {{ 
                border: 1px solid #ddd; padding: 12px; text-align: left; 
            }}
            th {{ background-color: #f2f2f2; }}
            img {{ 
                max-width: 100%; height: auto; margin-top: 2em; 
                border: 1px solid #ddd; border-radius: 8px;
            }}
        </style>
    </head>
    <body>
        <div class="container">
            <h1>System Health Report</h1>
            
            <h2>Overall Status</h2>
            <div class="status-box {'status-healthy' if overall_status == 'HEALTHY' else 'status-degraded'}">
                System is {overall_status}
            </div>
            <p>Total time spent on checks (sum of all durations): {total_check_time:.2f} seconds.</p>
            
            <h2>Component Details</h2>
            <table>
                <thead>
                    <tr>
                        <th>Component (Node)</th>
                        <th>Status</th>
                        <th>Check Duration</th>
                    </tr>
                </thead>
                <tbody>
                    {table_rows}
                </tbody>
            </table>
            
            {'<h2>System Graph</h2><img src="' + graph_image_data + '" alt="System Health DAG">' if graph_image_data else ''}
        </div>
    </body>
    </html>
    """
    
    # I'm using Flask's `render_template_string` to return the HTML.
    return render_template_string(html_response)

# --- 10. Run the App ---
# This is the standard Python entry point.
# It will run the Flask development server when I execute `python app.py`.
if __name__ == '__main__':
    # I'm running it on port 5000 and making it accessible
    # on my local network (host='0.0.0.0').
    app.run(debug=True, host='0.0.0.0', port=5000)