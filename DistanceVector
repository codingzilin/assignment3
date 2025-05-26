#!/usr/bin/env python3

import sys
from collections import defaultdict

class DistanceVector:
    def __init__(self):
        self.routers = set()
        self.links = {}  # (router1, router2) -> cost
        self.distance_tables = {}  # router -> {destination -> {next_hop -> cost}}
        self.routing_tables = {}  # router -> {destination -> (next_hop, cost)}
        self.INF = float('inf')
    
    def add_router(self, router):
        """Add a router to the network"""
        self.routers.add(router)
        if router not in self.distance_tables:
            self.distance_tables[router] = {}
        if router not in self.routing_tables:
            self.routing_tables[router] = {}
    
    def add_link(self, router1, router2, cost):
        """Add or update a link between two routers"""
        if cost == -1:
            # Remove link
            self.links.pop((router1, router2), None)
            self.links.pop((router2, router1), None)
        else:
            self.links[(router1, router2)] = cost
            self.links[(router2, router1)] = cost
            # Add routers if they don't exist
            self.add_router(router1)
            self.add_router(router2)
    
    def get_neighbors(self, router):
        """Get all neighbors of a router"""
        neighbors = set()
        for (r1, r2), cost in self.links.items():
            if r1 == router and cost != -1:
                neighbors.add(r2)
        return neighbors
    
    def get_link_cost(self, router1, router2):
        """Get the cost of direct link between two routers"""
        return self.links.get((router1, router2), self.INF)
    
    def initialize_distance_tables(self):
        """Initialize distance tables for all routers"""
        for router in sorted(self.routers):
            self.distance_tables[router] = {}
            for dest in sorted(self.routers):
                if dest != router:
                    self.distance_tables[router][dest] = {}
                    # Initialize with direct link costs
                    for neighbor in sorted(self.routers):
                        if neighbor != router:
                            if neighbor == dest:
                                # Direct link to destination
                                cost = self.get_link_cost(router, dest)
                                self.distance_tables[router][dest][neighbor] = cost
                            else:
                                # Via neighbor
                                self.distance_tables[router][dest][neighbor] = self.INF
    
    def print_distance_table(self, router, step):
        """Print distance table for a router at given step"""
        print(f"Distance Table of router {router} at t={step}:")
        
        # Get all destinations (all routers except current)
        destinations = sorted([r for r in self.routers if r != router])
        next_hops = sorted([r for r in self.routers if r != router])
        
        if not destinations:
            print()
            return
        
        # Print header
        header = "     " + "    ".join(f"{dest:<4}" for dest in destinations)
        print(header)
        
        # Print rows
        for next_hop in next_hops:
            row = f"{next_hop:<4} "
            for dest in destinations:
                if dest in self.distance_tables[router] and next_hop in self.distance_tables[router][dest]:
                    cost = self.distance_tables[router][dest][next_hop]
                    cost_str = "INF" if cost == self.INF else str(cost)
                else:
                    cost_str = "INF"
                row += f"{cost_str:<4} "
            print(row)
        print()
    
    def print_routing_table(self, router):
        """Print routing table for a router"""
        print(f"Routing Table of router {router}:")
        
        destinations = sorted([r for r in self.routers if r != router])
        for dest in destinations:
            if dest in self.routing_tables[router]:
                next_hop, cost = self.routing_tables[router][dest]
                if cost == self.INF:
                    print(f"{dest},INF,INF")
                else:
                    print(f"{dest},{next_hop},{cost}")
            else:
                print(f"{dest},INF,INF")
        print()
    
    def update_routing_table(self, router):
        """Update routing table based on distance table"""
        self.routing_tables[router] = {}
        
        for dest in self.distance_tables[router]:
            min_cost = self.INF
            best_next_hop = None
            
            # Find minimum cost path, choosing alphabetically first if tie
            for next_hop in sorted(self.distance_tables[router][dest].keys()):
                cost = self.distance_tables[router][dest][next_hop]
                if cost < min_cost:
                    min_cost = cost
                    best_next_hop = next_hop
            
            if best_next_hop is not None:
                self.routing_tables[router][dest] = (best_next_hop, min_cost)
    
    def run_distance_vector(self):
        """Run the distance vector algorithm until convergence"""
        self.initialize_distance_tables()
        
        step = 0
        changed = True
        
        while changed:
            # Print current distance tables
            for router in sorted(self.routers):
                self.print_distance_table(router, step)
            
            if step == 0:
                # First iteration: initialize with neighbor information
                for router in sorted(self.routers):
                    neighbors = self.get_neighbors(router)
                    for dest in self.distance_tables[router]:
                        for next_hop in self.distance_tables[router][dest]:
                            if next_hop in neighbors:
                                # Cost via this neighbor
                                link_cost = self.get_link_cost(router, next_hop)
                                if next_hop == dest:
                                    # Direct link
                                    self.distance_tables[router][dest][next_hop] = link_cost
                                else:
                                    # Will be updated in next iterations
                                    self.distance_tables[router][dest][next_hop] = self.INF
                            else:
                                self.distance_tables[router][dest][next_hop] = self.INF
                step += 1
                continue
            
            # Check for changes
            changed = False
            new_tables = {}
            
            for router in self.routers:
                new_tables[router] = {}
                for dest in self.distance_tables[router]:
                    new_tables[router][dest] = {}
                    for next_hop in self.distance_tables[router][dest]:
                        new_tables[router][dest][next_hop] = self.distance_tables[router][dest][next_hop]
            
            # Update distance tables using Bellman-Ford equation
            for router in sorted(self.routers):
                neighbors = self.get_neighbors(router)
                
                for dest in sorted(self.routers):
                    if dest == router:
                        continue
                    
                    for next_hop in sorted(self.routers):
                        if next_hop == router:
                            continue
                        
                        if next_hop in neighbors:
                            # Calculate cost via this neighbor
                            link_cost = self.get_link_cost(router, next_hop)
                            
                            if next_hop == dest:
                                # Direct link
                                new_cost = link_cost
                            else:
                                # Cost via neighbor: c(router, next_hop) + D_next_hop(dest)
                                if dest in self.routing_tables.get(next_hop, {}):
                                    neighbor_cost = self.routing_tables[next_hop][dest][1]
                                    new_cost = link_cost + neighbor_cost
                                else:
                                    new_cost = self.INF
                            
                            if new_cost != new_tables[router][dest][next_hop]:
                                new_tables[router][dest][next_hop] = new_cost
                                changed = True
                        else:
                            new_tables[router][dest][next_hop] = self.INF
            
            # Update distance tables and routing tables
            self.distance_tables = new_tables
            for router in sorted(self.routers):
                self.update_routing_table(router)
            
            step += 1
            
            # Prevent infinite loops
            if step > 50:
                break
        
        # Print final routing tables
        for router in sorted(self.routers):
            self.print_routing_table(router)

def main():
    dv = DistanceVector()
    
    # Read router names
    while True:
        line = input().strip()
        if line == "START":
            break
        dv.add_router(line)
    
    # Read initial topology
    while True:
        line = input().strip()
        if line == "UPDATE":
            break
        parts = line.split()
        router1, router2, cost = parts[0], parts[1], int(parts[2])
        dv.add_link(router1, router2, cost)
    
    # Run algorithm and print results
    dv.run_distance_vector()
    
    # Read updates
    updates_exist = False
    while True:
        line = input().strip()
        if line == "END":
            break
        updates_exist = True
        parts = line.split()
        router1, router2, cost = parts[0], parts[1], int(parts[2])
        dv.add_link(router1, router2, cost)
    
    # If there were updates, run algorithm again
    if updates_exist:
        dv.run_distance_vector()

if __name__ == "__main__":
    main()
