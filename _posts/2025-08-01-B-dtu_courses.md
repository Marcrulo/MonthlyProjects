---
title: 6. DTU course discovery 
description: Create a chrome extension for the DTU course website
published: true
---

## [](#prologue)Prologue
What started out as a curiosity, turned into an exam project in the DTU course "***Social Graphs and Interactions***" (02805). Repo can be found [here](https://github.com/ThereseStein/CourseAnalyzer). Even though we managed to create and analyse the course graph for the exam (*there was more depth to the project, but let's omit that for now*), I knew that I wanted to somehow help other students in practice, using this data. 

![big_graph]({{ site.baseurl }}/assets/images/courses/big_graph.png "big_graph")

*The graph shows all courses (which have neighbors), and how they are connected to other courses by their prerequisites, where each node is colored by department, and has size equivalent to its degree.*


Therefore I came up with a way to help fellow students, using a [Chrome extension that alters the DTU course website](https://chromewebstore.google.com/detail/dtu-extended-course-overv/pfgokeibjgebafnamhbgkgfgbmhmgpde) (and the GitHub [here](https://github.com/Marcrulo/DTU-courses-extension)). A problem with the current [website](https://kurser.dtu.dk/course/02285) is, that while it does show what courses are required for taking 'this' course (the "prerequisites"), it does not show which courses it is a prerequisite to. In other words, it does not show the "subsequent" courses. In general is it quite hard to get an overview of how courses are connected... but the extension solves that!

Let's go through the steps on by one, to see how it all comes together


## [](#scraping)Scraping
There is no API for retrieving course data nicely, which is why we need to scrape the course website instead (for each course). Using the **BeautifulSoup4** library we can easily go through all possible course number, and check if the website returns a valid course description:

```python
# Imports
import requests
import json
from bs4 import BeautifulSoup

# Initial
base_url = "https://kurser.dtu.dk/course/"
cookies = {
    'ASP.NET_SessionId' : "<SessionId>",   # not that important
    '{DTUCoursesPublicLanguage}' : 'en-GB' # ensure that the site is always in English
}

# Department number-to-name mapping
with open('department_names.json', 'r') as file:
    departments = json.load(file)
departments_keys = list(departments.keys())

# Loop through all possible course sites
valid_courses = {}
for dep in departments_keys:
    for i in range(0,1000):
        course_num = f"{dep}{i:03}"
        course_url = base_url + course_num
        response = requests.get(course_url, cookies=cookies)
        soup = BeautifulSoup(response.content, 'html.parser')
        title = soup.title.string
        if title is None:
            continue
        valid_courses[course_num] = response.text  

# Save text content for each course description
with open('valid_courses.json', 'w') as file:
    json.dump(valid_courses, file)

```

## [](#graphs)Course Graphs
Now we need to extract the relevant information from the HTML pages in order to create a graph, using the **NetworkX** library.

We start by creating all the nodes:
```python
# Initialize directd graph
G = nx.DiGraph()

# Go through each course
for course_num in valid_courses:
    department = course_num[:2] 
    G.add_node(course_num,
               course_num      = course_num,
               page            = valid_courses[course_num],
               department      = department, 
               color           = department_colors[department],
               department_name = department_names[department])
```

Then we add directed edge that points *away* from the preresuisite course:
```python
for course_num in valid_courses:

    # Initialize BeautifulSoup object
    page = G.nodes[course_num]['page']
    soup = BeautifulSoup(page, 'html.parser')

    # Define the search pattern to match both "Academic prerequisites" and "Mandatory prerequisites"
    search_pattern = r"(Academic prerequisites|Mandatory Prerequisites)"
    
    # Find the label element that matches the pattern
    label = soup.find('label', string=re.compile(search_pattern))
    if label is None:
        continue  # Skip if no label is found
    
    # Get the parent element that contains the label and prerequisites
    parent = label.find_parent().find_parent()
    
    # Get the second <td> (assuming it contains the prerequisites text)
    prerequisite = parent.find_all('td')[1].text

    # Remove whitespace and line breaks
    prerequisite = prerequisite.replace('\r', ' ').replace('\n', ' ')

    # Extract 5-digit course numbers
    prerequisites = set(re.findall(r'\d{5}', prerequisite))
    
    # Add edges to the graph for valid prerequisites
    for prerequisite in prerequisites:
        if prerequisite in G.nodes:
            if prerequisite != course_num:  # Skip self-loops
                G.add_edge(prerequisite, course_num)
```

And the most important step is then to create a subgraph for each course number, where the course of interest is the center or root, and each layer/level "forward" is the next level of subsequent courses, and each layer/level "backwards" is the next level of prerequisite courses. 


```python
# Get all courses
all_course_nums = list(G.nodes())

# Create subgraph for all course numbers
for center_node in all_course_nums:

    # Get forward and reverse BFS nodes
    forward_nodes = nx.single_source_shortest_path_length(G, center_node)
    reverse_nodes = nx.single_source_shortest_path_length(G.reverse(copy=False), center_node)

    # Combine all relevant nodes
    all_nodes = set(forward_nodes.keys()) | set(reverse_nodes.keys())
    levels = {}

    for node in all_nodes:
        if node == center_node:
            levels[node] = 0
        elif node in reverse_nodes:
            levels[node] = -reverse_nodes[node]
        else:
            levels[node] = forward_nodes[node]

    # Create a new DiGraph with only those edges
    filtered_subG = nx.DiGraph()
    filtered_subG.add_nodes_from(all_nodes)
    

    # Build layout positions
    pos = {}
    level_nodes = {}
    for node, level in levels.items():
        level_nodes.setdefault(level, []).append(node)

    for level in sorted(level_nodes):
        nodes = level_nodes[level]
        for i, node in enumerate(nodes):
            pos[node] = (level, -i)

    # Remove edges if they don't point "forward"
    edges = [
        (u, v) for u, v in G.subgraph(all_nodes).edges()
        if pos[u][0] < pos[v][0]
    ]
    filtered_subG.add_edges_from(edges)

    # Write to JSON
    graph_to_json(center_node, filtered_subG, levels)
```
(*the rest of the code can be found in the [GitHub repo](https://github.com/Marcrulo/DTU-courses-extension)*)

To see what this looks like, consider the image below. The center is **02180**. Is has 5 prerequisites: (**02100**,**02312**,**02105**,**01019**,**01017**), and is itself the prerequisite to 3 courses (**02256**,**02287**,**02285**)

![layer_graph]({{ site.baseurl }}/assets/images/courses/layer_graph.png "layer_graph")

The goal is to insert this graph into the course website itself, and this is where we get to the ***Chrome Extension***

## [](#chrome-extension)Chrome Extension
I quickly knew that it would be a problem to simply paste the image of the graph into the website. Not only is an image not interactive, but it would require quite a lot of data storage and loading.

Instead, I realized that the layered graph almost looked like a table structure, which is one of the most basic elements in HTML. Columns would indicate the "level" and rows would be added based on the number of nodes within the levels altogether.

### Practical Stuff
To start creating a Chrome extension, I simply followed [this tutorial](https://developer.chrome.com/docs/extensions/get-started/tutorial/hello-world). In essence, I the center of the project, `manifest.json` 

```json
{
  "name": "DTU - Extended Course Overview",
  "description": "",
  "version": "1.0",
  "manifest_version": 3,
  "action": {
    "default_popup": "popup.html"
  },
   "content_scripts": [
        {
        "js": ["anseki-leader-line/leader-line.min.js","content.js"],
        "css": ["my-css.css"],
        "matches": ["https://kurser.dtu.dk/course/*"]
        }
    ],
    "web_accessible_resources": [
    {
      "resources": ["graphs.json", "id_to_name.json"],
      "matches": ["<all_urls>"]
    }
  ],
    "icons": {
    "16": "icons/icon16.png",
    "32": "icons/icon32.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }   
}
```

This basically defines the main settings, as well as which files to import as well (javascript and css)

When you have a project folder with this file within it, you can start testing the extension locally. To get it public, you first need to pay the "Chrome Web Store developer registration fee", and then upload the project and get it approved by Google


### The Magic of HTML/CSS/JS
To alter the HTML content of a page, you first need to define that you need access to a certain page. The wider the permission required, the harder it might be to get the extension approved. Luckily, I only need permission to alter these pages:
> https://kurser.dtu.dk/course/*

We then need the `content.js` file that does the logic and rendering. The code is basically split into 7 sections:

**1** - Load relevant files

```javascript
// Get course ID from URL
// Example: https://kurser.dtu.dk/course/<course-id>
const courseId = window.location.pathname.split('/').pop();

// Load id_to_name mapping from JSON file
async function loadIdToName() {
  try {
    const response = await fetch(chrome.runtime.getURL("id_to_name.json"));
    return await response.json();
  } catch (err) {
    console.error("Error loading id_to_name.json", err);
    throw err; // rethrow so the caller knows it failed
  }
}

// Load graph data from JSON file for a specific courseId
async function loadGraph(courseId) {
  try {
    const response = await fetch(chrome.runtime.getURL("graphs.json"));
    const data = await response.json();
    return data[courseId];
  } catch (err) {
    console.error("Error loading graphs.json", err);
    throw err;
  }
}
```

**2** - Run code as soon as files are loaded (using async)

```javascript
(async () => {
  const graph = await loadGraph(courseId);
  const mapping = await loadIdToName();
  
  ...

})();
```
**3** - Extend left-most div/table (the element with course type, name, points etc.)

```javascript
// Insert new rows into the second table on the page
const tables = document.querySelectorAll("table");
const table = tables[1];

const rowBefore = table.insertRow();
const cell_before1 = rowBefore.insertCell();
const cell_before2 = rowBefore.insertCell();

// Disclaimer row
const rowDisclaimer = table.insertRow();
const cell_disclaimer = rowDisclaimer.insertCell();
cell_disclaimer.colSpan = 2;
cell_disclaimer.innerHTML =
`<label style='font-style: italic; color: gray;'>
    The graph shows all possible paths of courses leading to/from ${courseId}. <br>
    However, it is rarely required to take all the courses shown!
</label>`;

const rowPrereq = table.insertRow();
const cell_title_prereq = rowPrereq.insertCell();
const cell_content_prereq = rowPrereq.insertCell();

const rowSubseq = table.insertRow();
const cell_title_subseq = rowSubseq.insertCell();
const cell_content_subseq = rowSubseq.insertCell();

// Footer / contact row
const rowContact = table.insertRow();
const cell_contact = rowContact.insertCell();
cell_contact.colSpan = 2;
cell_contact.innerHTML =
`<label style='font-style: italic; color: gray; font-size: 8px;'>
    <br><br>You are welcome to contribute to the project by <br>
    leaving feedback or suggesting improvements: <br>
    <a href="https://github.com/Marcrulo/DTU-courses-extension" target="_blank">https://github.com/Marcrulo/DTU-courses-extension</a>
</label>`;

const rowAfter = table.insertRow();
const cell_after1 = rowAfter.insertCell();
const cell_after2 = rowAfter.insertCell();

// Section titles
cell_title_prereq.innerHTML = "<label>Prerequisite course <br> paths</label>";
cell_title_subseq.innerHTML = "<label>Subsequent course <br> paths</label>";

// Section dividers
[cell_before1, cell_before2, cell_after1, cell_after2].forEach(cell => {
         cell.innerHTML = `<div style="border-top: 1px solid #b50404; margin: 10px 0;"></div>`;
    });     

```

**4** - Constructing tables structure, where the cells represent graph nodes (courses)

```javascript
/* ====== BUILD TABLES ====== */
function buildTables(courseId, graph) {
const { max_subseq = 0, max_prereq = 0, subseq_height = 0, prereq_height = 0 } = graph;

const table_prereq = Array.from({ length: prereq_height }, () =>
    Array(max_prereq + 1).fill(null)
);
const table_subseq = Array.from({ length: subseq_height }, () =>
    Array(max_subseq + 1).fill(null)
);

const prereq_row = {};
const subseq_row = {};

// Place nodes in correct table depending on level
for (const { id, level } of graph.nodes) {
    if (level < 0) {
    if (!(level in prereq_row)) prereq_row[level] = 0;
    table_prereq[prereq_row[level]][max_prereq + level] = id;
    prereq_row[level]++;
    } else if (level > 0) {
    if (!(level in subseq_row)) subseq_row[level] = 0;
    table_subseq[subseq_row[level]][level] = id;
    subseq_row[level]++;
    }
}

// Place the courseId itself
if (prereq_height > 0) table_prereq[0][max_prereq] = courseId;
if (subseq_height > 0) table_subseq[0][0] = courseId;

// Ensure non-empty tables
if (table_prereq.length === 0) table_prereq.push([null]);
if (table_subseq.length === 0) table_subseq.push([null]);

// Add spacing columns
for (let row of table_prereq) {
    for (let j = 1; j < row.length; j += 2) row.splice(j, 0, null);
}
for (let row of table_subseq) {
    for (let j = 1; j < row.length; j += 2) row.splice(j, 0, null);
}

// Helper: sort values in each column alphabetically
function sortColumns(table) {
    const rows = table.length;
    const cols = table[0]?.length || 0;

    for (let col = 0; col < cols; col++) {
        const values = [];
        for (let row = 0; row < rows; row++) {
            if (table[row][col] !== null) values.push(table[row][col]);
        }
        values.sort();
        for (let row = 0; row < rows; row++) {
            table[row][col] = row < values.length ? values[row] : null;
        }
    }
}

// Helper: center values vertically in each column
function centerColumns(table) {
    const rows = table.length;
    const cols = table[0]?.length || 0;

    for (let col = 0; col < cols; col++) {
        const values = [];
        for (let row = 0; row < rows; row++) {
            if (table[row][col] !== null) values.push(table[row][col]);
        }
        const filled = values.length;
        if (filled === 0) continue;

        const topPadding = Math.floor((rows - filled) / 2);
        for (let row = 0; row < rows; row++) {
            const idx = row - topPadding;
            table[row][col] = idx >= 0 && idx < filled ? values[idx] : null;
        }
    }
}

sortColumns(table_prereq);
sortColumns(table_subseq);
centerColumns(table_prereq);
centerColumns(table_subseq);

return { table_prereq, table_subseq };
}

const { table_prereq, table_subseq } = buildTables(courseId, graph);
```

**5** - Render tables in HTML

```javascript
/* ====== RENDER TABLES ====== */
  function renderTable(table, type, fontSizeBase) {
    return `
      <table id='course-overview-${type}' style="width: 100%; border-collapse: separate;">
        ${table.map(row => `
          <tr>
            ${row.map((cell, index) => `
              <td id='${cell ? cell + "_" + type : ""}'
                  style="width: ${index % 2 === 0 ? "30px" : "50px"};">
                ${cell ? `
                  <a href="https://kurser.dtu.dk/course/${cell}"
                     class="tooltip-link"
                     style="color: #b50404; text-decoration: none; font-size: ${fontSizeBase - (type === "prereq" ? graph.max_prereq : graph.max_subseq)}px;">
                    ${cell}
                    <span class="tooltip-text">${mapping[cell] || "No info available"}</span>
                  </a>` : ""}
              </td>`).join("")}
          </tr>`).join("")}
      </table>
    `;
  }

  // Fill content cells
  cell_content_prereq.innerHTML = renderTable(table_prereq, "prereq", 16);
  cell_content_subseq.innerHTML = renderTable(table_subseq, "subseq", 16);

  // Replace with labels if empty
  function isTableEmpty(table) {
    return table.every(row => row.every(cell => cell === null));
  }
  if (isTableEmpty(table_prereq)) {
    cell_content_prereq.innerHTML =
      `<label style="font-style: italic; color: gray;">This course does not have any prerequisites</label>`;
  }
  if (isTableEmpty(table_subseq)) {
    cell_content_subseq.innerHTML =
      `<label style="font-style: italic; color: gray;">This course does not lead to any other courses</label>`;
  }

  // Highlight "endpoints" of prereq/subseq tables
  cell_content_prereq.querySelectorAll("td:last-child").forEach(cell => {
    cell.style.fontStyle = "italic";
    cell.style.textDecoration = "underline";
    cell.style.color = "#b50404";
  });
  cell_content_subseq.querySelectorAll("td:first-child").forEach(cell => {
    cell.style.fontStyle = "italic";
    cell.style.textDecoration = "underline";
    cell.style.color = "#b50404";
  });
```

**6** - Draw lines (edges) between cells, corresponding to the graph we computed. Lines are created using the "LeaderLine" package

```javascript
/* ====== DRAW STATIC LEADERLINES ====== */
  for (const edge of graph.edges) {
    for (const suffix of ["_prereq", "_subseq"]) {
      const startNode = document.getElementById(edge.source + suffix);
      const endNode = document.getElementById(edge.target + suffix);
      if (startNode && endNode) {
        new LeaderLine(startNode, endNode, {
          size: 2,
          color: "rgba(36, 4, 9, 0.3)",
          path: "straight",
          endPlug: "arrow3",
          startSocket: "right",
          endSocket: "left",
        });
      }
    }
  }
```

**7** - Upon hovering, highlight node and its 1-hop neighborhood (highlight immediate neighbors in both directions)

```javascript
/* ====== INTERACTIVE HIGHLIGHTING ====== */
let tempLines = [];
let highlightedLinks = [];

document.querySelectorAll(".tooltip-link").forEach(link => {
link.addEventListener("mouseenter", () => {
    const table_id = "_" + link.closest("table").id.split("-")[2]; // "_prereq" or "_subseq"
    const hoveredId = link.href.split("/").pop();

    // Find edges connected to hovered course
    const connectedEdges = graph.edges.filter(
    e => e.source === hoveredId || e.target === hoveredId
    );

    // Collect all connected IDs
    const connectedIds = new Set([hoveredId]);
    connectedEdges.forEach(e => {
    connectedIds.add(e.source);
    connectedIds.add(e.target);
    });

    // Highlight connected nodes
    connectedIds.forEach(id => {
    const elems = document.querySelectorAll(`td[id='${id}${table_id}'] .tooltip-link`);
    elems.forEach(el => {
        el.style.fontWeight = "bold";
        el.style.textDecoration = "underline";
        el.style.color = "#b50404";
        highlightedLinks.push(el);
    });
    });

    // Draw temporary leaderlines
    connectedEdges.forEach(edge => {
    const startNode = document.getElementById(edge.source + table_id);
    const endNode = document.getElementById(edge.target + table_id);
    if (startNode && endNode) {
        const line = new LeaderLine(startNode, endNode, {
        size: 3,
        color: "rgba(181, 4, 4, 0.9)",
        path: "straight",
        endPlug: "arrow3",
        startSocket: "right",
        endSocket: "left",
        });
        tempLines.push(line);
    }
    });
});

link.addEventListener("mouseleave", () => {
    // Remove temporary lines
    tempLines.forEach(line => line.remove());
    tempLines = [];

    // Remove highlights
    highlightedLinks.forEach(el => {
    el.style.fontWeight = "";
    el.style.textDecoration = "";
    el.style.color = "";
    });
    highlightedLinks = [];
  });
});
```

In essence, we add a section to the course website, and create a table that represents the course graph. I use the awesome ***LeaderLine*** library for creating arrows (*directed edges*) between cell elements (*nodes*)

The result looks something like this:

![screenshot]({{ site.baseurl }}/assets/images/courses/screenshot.png "screenshot")



## [](#maintenance)Maintenance 
As of now, the extension works fine, but this is only the case for the current course structure. If anything changes with the courses or the prerequisites, the graph will technically be wrong, which may cause confusion and frustration. Let's try and fix that.

### Automatic updates
I know for a fact that the course structure is changed once a year at around May or June. If I simply run the script yearly in July, then all will be good, right?

What will likely happen is that I forget about the project, or maybe don't really care anymore. Maybe the first years it will work fine, but after that it will not work anymore. 

The obvious solution for this is then to create a scheduled job for scraping the course websites and create the graph. I tried some recommended websites for free python script hosting, but I also need file hosting as well. It did not go well. I then found that **GitHub actions** lets you do exactly that. GitHub actions is usually used for CI/CD tasks, but it doesn't *have* to be that. 

I have created an `update_courses.yml` workflow file. This will run all the processing once a year on July 1st.

```yaml
name: Update Graph

on:
  schedule:
    # Runs at 00:00 UTC on July 1st every year
    - cron: '0 0 1 7 *'
  workflow_dispatch: # allows manual run

jobs:
  run-python:
    runs-on: ubuntu-latest

    permissions:
      contents: write

    steps:
    - name: Checkout repository
      uses: actions/checkout@v3
      with:
        token: ${{ secrets.GITHUB_TOKEN }}

    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.11'

    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        if [ -f requirements.txt ]; then pip install -r requirements.txt; fi

    - name: Run Python scripts
      run: |
        cd processing
        python 1_scrape_content.py
        python 2_create_graph.py

    - name: Push output file to branch
      run: |
        git config --global user.name "github-actions[bot]"
        git config --global user.email "github-actions[bot]@users.noreply.github.com"
        git add jsons/valid_courses.json
        git add jsons/id_to_name.json
        git add jsons/graphs.json
        git commit -m "Update output for $(date +'%Y-%m-%d')"
        git push -u origin master --force

```

The coolest thing is that you let a "bot user" push files to the repo. So instead of adding the graph JSON file directly into the Chrome extension, the file will be located and updated at a specific file in the repo, which is directly accessible from the web (since its a public repo). It will basically act as a GitHub "gist" file.

### Open-Sourcing
To be realistic, I can't assume that people will agree with the design of the extension, so I want to give people the option to add improvements to it. It is a gift for the public after all. In the event that the course website undergoes a drastic change that renders the extension useless, I want people in the future to be able to fix that as well. I will, of course, be responsible for reviewing pull requests.

Maybe no one cares. Who knows? Now that I have this tool, I can't imagine a world without it anymore. I want everyone to get the same benefits as I got from it. It was a fun project nonetheless.



