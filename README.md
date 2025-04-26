# Import necessary libraries
import networkx as nx
import folium
import pandas as pd
import matplotlib.pyplot as plt

# Example delivery points (latitude, longitude)
delivery_points = [
    (40.712776, -74.005974),  # New York City
    (34.052235, -118.243683),  # Los Angeles
    (41.878113, -87.629799),   # Chicago
    (51.507351, -0.127758),    # London
]

# Function to simulate traffic congestion
def simulate_traffic_condition(time_of_day):
    if time_of_day in ['8-9AM', '5-6PM']:  # Peak hours
        return 1.5  # 50% more time due to traffic
    elif time_of_day in ['12-1PM']:  # Moderate traffic
        return 1.2  # 20% more time
    else:
        return 1  # No additional time in off-peak hours

# Function to calculate travel time (example using Euclidean distance)
def calculate_travel_time(point1, point2):
    lat1, lon1 = point1
    lat2, lon2 = point2
    return ((lat2 - lat1)**2 + (lon2 - lon1)**2)**0.5

# Create a graph
G = nx.Graph()

# Add delivery points (nodes)
for i, point in enumerate(delivery_points):
    G.add_node(i, pos=point)

# Add edges between delivery points with travel times adjusted for traffic
for i in range(len(delivery_points)):
    for j in range(i + 1, len(delivery_points)):
        travel_time = calculate_travel_time(delivery_points[i], delivery_points[j])
        # Adjust travel time based on traffic congestion
        adjusted_travel_time = travel_time * simulate_traffic_condition('8-9AM')  # Example: Peak hour
        G.add_edge(i, j, weight=adjusted_travel_time)

# Use TSP approximation to find the shortest route
optimized_route = nx.approximation.traveling_salesman_problem(G, cycle=False, weight='weight')

# Visualize the optimized route using folium
m = folium.Map(location=[delivery_points[0][0], delivery_points[0][1]], zoom_start=12)

# Add markers for each delivery point
for point in [delivery_points[i] for i in optimized_route]:
    folium.Marker([point[0], point[1]]).add_to(m)

# Add the optimized route as a polyline
route_coords = [delivery_points[i] for i in optimized_route]
folium.PolyLine(locations=route_coords, color='blue').add_to(m)

# Save the map as an HTML file
m.save("optimized_route_map.html")

# Calculate and display the total travel time
total_travel_time = sum([G[optimized_route[i]][optimized_route[i+1]]['weight'] for i in range(len(optimized_route)-1)])
print(f"Total travel time for optimized route: {total_travel_time} units")

# Plot a simple cost vs. time graph
times = [G[optimized_route[i]][optimized_route[i+1]]['weight'] for i in range(len(optimized_route)-1)]
plt.plot(range(len(times)), times, label='Travel Time')
plt.xlabel('Route Segments')
plt.ylabel('Travel Time (units)')
plt.title('Travel Time per Route Segment')
plt.legend()
plt.show()

