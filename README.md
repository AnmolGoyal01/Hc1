class Graph {
  constructor(links, banks) {
    this.adjacencyList = new Map();
    this.costs = new Map();

    banks.forEach(({ bic, charge }) => {
      this.costs.set(bic, charge);
      this.adjacencyList.set(bic, []);
    });

    links.forEach(({ fromBic, toBic, time }) => {
      this.adjacencyList.get(fromBic).push({ node: toBic, weight: time });
      this.adjacencyList.get(toBic).push({ node: fromBic, weight: time });
    });
  }

  dijkstra(start, end, weightKey) {
    const pq = new Map();
    const distances = new Map();
    const previous = new Map();

    this.adjacencyList.forEach((_, node) => {
      distances.set(node, Infinity);
      previous.set(node, null);
    });
    distances.set(start, 0);
    pq.set(start, 0);

    while (pq.size) {
      const [currentNode] = [...pq.entries()].sort((a, b) => a[1] - b[1]);
      pq.delete(currentNode[0]);

      if (currentNode[0] === end) break;
      for (const neighbor of this.adjacencyList.get(currentNode[0])) {
        const weight = weightKey === 'time' ? neighbor.weight : this.costs.get(neighbor.node);
        const newDistance = distances.get(currentNode[0]) + weight;
        if (newDistance < distances.get(neighbor.node)) {
          distances.set(neighbor.node, newDistance);
          previous.set(neighbor.node, currentNode[0]);
          pq.set(neighbor.node, newDistance);
        }
      }
    }
    return this.constructPath(previous, start, end, distances.get(end));
  }

  constructPath(previous, start, end, total) {
    const path = [];
    let current = end;
    while (current) {
      path.unshift(current);
      current = previous.get(current);
    }
    return {
      route: path.join(' -> '),
      [typeof total === 'number' ? 'time' : 'cost']: total,
    };
  }
}

function getShortestTimePath(fromBic, toBic, links, banks) {
  const graph = new Graph(links, banks);
  return graph.dijkstra(fromBic, toBic, 'time');
}

function cheapestPath(fromBic, toBic, links, banks) {
  const graph = new Graph(links, banks);
  return graph.dijkstra(fromBic, toBic, 'cost');
}

// Example Usage
const links = [
  { fromBic: 'A', toBic: 'B', time: 10 },
  { fromBic: 'B', toBic: 'C', time: 20 },
  { fromBic: 'A', toBic: 'C', time: 50 },
];

const banks = [
  { bic: 'A', charge: 5 },
  { bic: 'B', charge: 10 },
  { bic: 'C', charge: 2 },
];

console.log(getShortestTimePath('A', 'C', links, banks));
console.log(cheapestPath('A', 'C', links, banks));
