# Building-Internet-on-Misty-Mountains-
import java.util.ArrayList;
import java.util.Scanner;
import java.io.File;
import java.io.FileNotFoundException;

public class Prog2Run {
    public static void main(String[] args) {
        try {
            // Initialize Scanner with the file
            Scanner scanner = new Scanner(new File("p2_test.txt"));

            // Read the number of houses
            int n = scanner.nextInt();
            float[] satcost = new float[n + 1];
            satcost[0] = 0; // Initialize the first element to 0 for consistency

            // Read the satellite installation costs for each house
            for (int i = 1; i <= n; i++) {
                satcost[i] = scanner.nextFloat();
            }

            // Read the number of possible cable connections
            ArrayList<Link> llist = new ArrayList<Link>();
            int e = scanner.nextInt();
            for (int i = 0; i < e; i++) {
                int v1 = scanner.nextInt();
                int v2 = scanner.nextInt();
                float w = scanner.nextFloat();
                Link l = new Link(v1, v2, w);
                l.print(); // Print each link as it's read
                llist.add(l);
            }
            scanner.close();
            System.out.println();

            // Call the main algorithm method from Prog2 and store the result
            Prog2Return p2 = Prog2.Prog2(satcost, llist);

            // Print the satellite installations
            System.out.print("Satellite : ");
            for (int i = 1; i <= n; i++) {
                if (p2.sat[i]) {
                    System.out.print(i + " ");
                }
            }
            System.out.println();

            // Print the cable links in the MST
            System.out.print("Links : ");
            for (int i = 0; i < p2.links.size(); i++) {
                Link l = p2.links.get(i);
                l.fix();  // If fix() is needed to adjust the link, call it here
                l.print(); // Print each link in the MST
                System.out.print(" ");
            }
            System.out.println();

            // Print the path from house 1 to the satellite-connected house
            System.out.print("Path : ");
            for (int i = 0; (i < p2.path.length) && (p2.path[i] >= 0); i++) {
                System.out.print(p2.path[i] + " ");
            }
            System.out.println();

        } catch (FileNotFoundException e) {
            System.out.println("File not found: p2_test.txt");
            e.printStackTrace();
        }
    }
}


public class Link {
    
    int v1, v2;
    float w;

 
    public Link() {
	v1 = 0;
	v2 = 0;
	w = 0;
	}

    public Link(int x, int y, float z) {
	    v1 = x;
            v2 = y;
            w = z;
	}

    public void Set(int x, int y, float z) {
	    v1 = x;
            v2 = y;
            w = z;
	}

    public int v1() {
	return v1;
	}

    public int v2() {
	return v2;
	}

    public float w() {
	return w;
	}

    public void fix() {
	  if (v1 > v2) {
		int temp = v1;
		v1 = v2;
		v2 = temp;
	   }

	}

    public void print() {
	System.out.print("(" + v1 + " -- " + v2 + " : " + w + ")");


    }
}


import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;

public class MyGraph {

    private int numVertices;
    private ArrayList<Link> edges;

    // Constructor
    public MyGraph(int vertices) {
        this.numVertices = vertices;
        this.edges = new ArrayList<>();
    }

    // Adds an edge if it doesn’t already exist
    public boolean addEdge(int a, int b, float w) {
        if (a < 1 || a > numVertices || b < 1 || b > numVertices) return false;
        if (edgeExists(a, b)) return false;
        edges.add(new Link(a, b, w));
        return true;
    }

    // Check if an edge exists between two vertices
    private boolean edgeExists(int a, int b) {
        return edges.stream().anyMatch(edge -> (edge.v1() == a && edge.v2() == b) || (edge.v1() == b && edge.v2() == a));
    }

    // Constructs the MST using Kruskal's algorithm
    public ArrayList<Link> MST() {
        ArrayList<Link> mstLinks = new ArrayList<>();
        Collections.sort(edges, Comparator.comparingDouble(Link::w));

        UnionFind uf = new UnionFind(numVertices);

        for (Link edge : edges) {
            if (uf.union(edge.v1(), edge.v2())) {
                mstLinks.add(edge);
            }
        }
        return mstLinks;
    }

    // Union-Find helper class
    private class UnionFind {
        private int[] parent;
        private int[] size;

        public UnionFind(int n) {
            parent = new int[n + 1];
            size = new int[n + 1];
            for (int i = 1; i <= n; i++) {
                parent[i] = i;
                size[i] = 1;
            }
        }

        public int find(int i) {
            if (parent[i] != i) {
                parent[i] = find(parent[i]); // Path compression
            }
            return parent[i];
        }

        public boolean union(int x, int y) {
            int rootX = find(x);
            int rootY = find(y);
            if (rootX == rootY) return false;

            // Union by size
            if (size[rootX] < size[rootY]) {
                parent[rootX] = rootY;
                size[rootY] += size[rootX];
            } else {
                parent[rootY] = rootX;
                size[rootX] += size[rootY];
            }
            return true;
        }
    }
}


5
10.5 5.5 20.3 6.1 18.4
8
4 1  30.4
3 4  2.2
5 3  4.0
1 3  11.7
1 2  1.3
2 5  9.4
4 5  3.1
2 3   90
# satellite connect house 2 and 4, cables build: 1-2 3-4 4-5

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;
import java.util.LinkedList;
import java.util.Queue;

public class Prog2 {

    public static Prog2Return Prog2(float[] satcost, ArrayList<Link> L1) {
        int n = satcost.length - 1; // Number of houses (1-based index)

        // Initialize result object
        Prog2Return result = new Prog2Return(n);

        // Union-Find data structure to track connected components
        UnionFind uf = new UnionFind(n + 1); // Treat satellite as a connection from vertex 0

        // Step 1: Create a list of all edges (satellite connections and cable links)
        ArrayList<Item> costs = new ArrayList<>();
        for (int i = 1; i <= n; i++) {
            costs.add(new Item(0, i, satcost[i]));  // Satellite as an edge from vertex 0
        }
        for (Link link : L1) {
            costs.add(new Item(link.v1(), link.v2(), link.w()));  // Cable option as edge
        }

        // Step 2: Sort connections by ascending cost
        Collections.sort(costs, Comparator.comparingDouble(Item::getCost));

        // Step 3: Use a modified Kruskal's algorithm to find the cheapest way to connect all houses
        for (Item item : costs) {
            if (uf.allActive(n)) break;  // Stop if all houses are active

            if (item.house1 == 0) {  // Satellite edge
                int house = item.house2;
                if (!uf.isActive(house)) {
                    result.sat[house] = true; // Mark house as using satellite
                    uf.union(0, house);  // Connect house to satellite
                }
            } else {  // Cable connection
                int v1 = item.house1;
                int v2 = item.house2;
                if (uf.find(v1) != uf.find(v2)) { // If v1 and v2 are in different components
                    result.links.add(new Link(v1, v2, (float) item.cost));
                    uf.union(v1, v2); // Merge the components
                }
            }
        }

        // Step 4: Ensure each connected component has access to a satellite
        boolean[] visited = new boolean[n + 1];
        for (int i = 1; i <= n; i++) {
            if (!visited[i] && uf.isActive(i)) {
                ArrayList<Integer> component = new ArrayList<>();
                collectComponent(i, component, result.links, visited);

                boolean hasSatellite = component.stream().anyMatch(node -> result.sat[node]);
                if (!hasSatellite) { // No satellite in the component, assign one
                    int minCostNode = component.get(0);
                    float minCost = satcost[minCostNode];
                    for (int node : component) {
                        if (satcost[node] < minCost) {
                            minCost = satcost[node];
                            minCostNode = node;
                        }
                    }
                    result.sat[minCostNode] = true;
                }
            }
        }

        // Step 5: Perform BFS to determine the shortest path from node 1 to a satellite node
        ArrayList<Integer> pathList = new ArrayList<>();
        visited = new boolean[n + 1];
        bfs(1, pathList, result.links, visited, result.sat);
        for (int i = 0; i < pathList.size(); i++) {
            result.path[i] = pathList.get(i);
        }

        return result;
    }

    // Helper method to collect all nodes in a connected component
    private static void collectComponent(int node, ArrayList<Integer> component, ArrayList<Link> mstLinks, boolean[] visited) {
        visited[node] = true;
        component.add(node);

        for (Link link : mstLinks) {
            int neighbor = -1;
            if (link.v1() == node && !visited[link.v2()]) {
                neighbor = link.v2();
            } else if (link.v2() == node && !visited[link.v1()]) {
                neighbor = link.v1();
            }
            if (neighbor != -1) {
                collectComponent(neighbor, component, mstLinks, visited);
            }
        }
    }

    // BFS to find the shortest path from node 1 to a satellite node
    private static void bfs(int start, ArrayList<Integer> path, ArrayList<Link> mstLinks, boolean[] visited, boolean[] satelliteNodes) {
        Queue<Integer> queue = new LinkedList<>();
        queue.add(start);
        visited[start] = true;

        // Track the parent of each node to reconstruct the path
        int[] parent = new int[visited.length];
        parent[start] = -1;

        while (!queue.isEmpty()) {
            int node = queue.poll();

            // Stop as soon as we reach a satellite node
            if (satelliteNodes[node]) {
                // Reconstruct the path from the end node back to the start
                while (node != -1) {
                    path.add(0, node);  // Add each node to the start of the list
                    node = parent[node];
                }
                return;
            }

            // Traverse all connected nodes
            for (Link link : mstLinks) {
                int neighbor = -1;
                if (link.v1() == node && !visited[link.v2()]) {
                    neighbor = link.v2();
                } else if (link.v2() == node && !visited[link.v1()]) {
                    neighbor = link.v1();
                }
                if (neighbor != -1) {
                    visited[neighbor] = true;
                    queue.add(neighbor);
                    parent[neighbor] = node;
                }
            }
        }
    }

    // Union-Find class to manage connected components
    private static class UnionFind {
        private int[] parent;

        public UnionFind(int n) {
            parent = new int[n + 1];
            for (int i = 0; i <= n; i++) {
                parent[i] = i;
            }
        }

        public int find(int i) {
            if (parent[i] != i) {
                parent[i] = find(parent[i]);
            }
            return parent[i];
        }

        public void union(int i, int j) {
            int rootI = find(i);
            int rootJ = find(j);
            if (rootI != rootJ) {
                parent[rootI] = rootJ;
            }
        }

        public boolean isActive(int i) {
            return find(i) == find(0);
        }

        public boolean allActive(int n) {
            for (int i = 1; i <= n; i++) {
                if (find(i) != find(0)) return false;
            }
            return true;
        }
    }

    // Class representing an item (satellite or cable link) for sorting and processing
    private static class Item {
        int house1, house2;
        double cost;

        public Item(int house1, int house2, double cost) {
            this.house1 = house1;
            this.house2 = house2;
            this.cost = cost;
        }

        public double getCost() {
            return this.cost;
        }
    }
}

import java.util.ArrayList;

public class Prog2Return {
    
    public boolean [] sat;
    public ArrayList<Link> links;
    public int [] path;


    public Prog2Return(int n) {
	     sat = new boolean[n + 1];
             path = new int[n + 1];
             for (int i = 0; i <= n; i++) {
	            sat[i] = false;
 		    path[i] = -1;
		}
	     path[0] = 1;
             links = new ArrayList<Link>();
	}

    public boolean[] getSat() {
	return sat.clone();
    }

    public ArrayList<Link> getLinks() {
	return links;
    }

    public int[] getPath() {
	return path.clone();
    }

}


