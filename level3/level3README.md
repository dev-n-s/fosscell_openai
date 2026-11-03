#include <stdio.h>
#include <stdlib.h>
#include <limits.h>
#include <stdint.h>

struct Edge {
    int src, dest, weight;
};

struct Graph {
    int V, E;
    struct Edge *edge;
};

struct Graph *createGraph(int V, int E) {
    if (V <= 0 || E < 0 ||
        (size_t)E > SIZE_MAX / sizeof(struct Edge)) {
        return NULL;
    }

    struct Graph *graph = malloc(sizeof *graph);
    if (graph == NULL) {
        return NULL;
    }

    graph->V = V;
    graph->E = E;
    graph->edge = E == 0 ? NULL : malloc((size_t)E * sizeof *graph->edge);

    if (E != 0 && graph->edge == NULL) {
        free(graph);
        return NULL;
    }

    return graph;
}

static int addWouldOverflow(int a, int b) {
    return (b > 0 && a > INT_MAX - b) ||
           (b < 0 && a < INT_MIN - b);
}

static int validGraph(const struct Graph *graph) {
    if (graph == NULL || graph->V <= 0 || graph->E < 0 ||
        (graph->E > 0 && graph->edge == NULL)) {
        return 0;
    }

    for (int i = 0; i < graph->E; ++i) {
        if (graph->edge[i].src < 0 || graph->edge[i].src >= graph->V ||
            graph->edge[i].dest < 0 || graph->edge[i].dest >= graph->V) {
            return 0;
        }
    }

    return 1;
}

int BellmanFord(const struct Graph *graph, int src) {
    if (!validGraph(graph) || src < 0 || src >= graph->V) {
        fprintf(stderr, "Invalid graph or source vertex\n");
        return 0;
    }

    int *dist = malloc((size_t)graph->V * sizeof *dist);
    if (dist == NULL) {
        fprintf(stderr, "Could not allocate distances\n");
        return 0;
    }

    for (int i = 0; i < graph->V; ++i) {
        dist[i] = INT_MAX;
    }
    dist[src] = 0;

    for (int pass = 1; pass < graph->V; ++pass) {
        int changed = 0;

        for (int j = 0; j < graph->E; ++j) {
            int u = graph->edge[j].src;
            int v = graph->edge[j].dest;
            int weight = graph->edge[j].weight;

            if (dist[u] == INT_MAX) {
                continue;
            }
            if (addWouldOverflow(dist[u], weight)) {
                fprintf(stderr, "Distance exceeds int range\n");
                free(dist);
                return 0;
            }

            int candidate = dist[u] + weight;
            if (candidate < dist[v]) {
                dist[v] = candidate;
                changed = 1;
            }
        }

        if (!changed) {
            break;
        }
    }

    for (int i = 0; i < graph->E; ++i) {
        int u = graph->edge[i].src;
        int v = graph->edge[i].dest;
        int weight = graph->edge[i].weight;

        if (dist[u] != INT_MAX &&
            !addWouldOverflow(dist[u], weight) &&
            dist[u] + weight < dist[v]) {
            printf("Graph contains a negative-weight cycle reachable from source\n");
            free(dist);
            return 0;
        }
    }

    printf("Vertex Distance from Source\n");
    for (int i = 0; i < graph->V; ++i) {
        if (dist[i] == INT_MAX) {
            printf("%d\t\t unreachable\n", i);
        } else {
            printf("%d\t\t %d\n", i, dist[i]);
        }
    }

    free(dist);
    return 1;
}

int main(void) {
    const int V = 5;
    const int E = 8;
    struct Graph *graph = createGraph(V, E);

    if (graph == NULL) {
        fprintf(stderr, "Could not create graph\n");
        return EXIT_FAILURE;
    }

    graph->edge[0] = (struct Edge){0, 1, -1};
    graph->edge[1] = (struct Edge){0, 2, 4};
    graph->edge[2] = (struct Edge){1, 2, 3};
    graph->edge[3] = (struct Edge){1, 3, 2};
    graph->edge[4] = (struct Edge){1, 4, 2};
    graph->edge[5] = (struct Edge){3, 2, 5};
    graph->edge[6] = (struct Edge){3, 1, 1};
    graph->edge[7] = (struct Edge){4, 3, -3};

    int ok = BellmanFord(graph, 0);
    free(graph->edge);
    free(graph);

    return ok ? EXIT_SUCCESS : EXIT_FAILURE;
}
