---
tags:
  - CS
  - Project
时间: 2024-03-15T08:53:00
---
```python
# This is the definition part of the StationInfo class
class StationInfo:
	""""
	 StationInfo provides a central repository for station information.     
	 This class is designed to get the station name, ID, and neighbours.
	""""
    m_name_to_id_dict = {}
    m_id_to_name_dict = {}
    m_id_neighbours = {}

    @classmethod
    def LoadData(cls, filename):
        with open(filename, 'r') as file:
            for line in file:
                parts = line.strip().split('|')
                if len(parts) >= 3:  # Ensure there are at least three parts in the line
                    station_id, station_name, neighbours = parts[0], parts[1], parts[2]
                    cls.m_name_to_id_dict[station_name] = station_id
                    cls.m_id_to_name_dict[station_id] = station_name
                    cls.m_id_neighbours[station_id] = neighbours.split(',')
    
    @classmethod
    def GetNameFromID(cls, id):
        return cls.m_id_to_name_dict.get(id, "Station ID not found")

    @classmethod
    def GetIDFromName(cls, name):
        return cls.m_name_to_id_dict.get(name, "Station name not found")

    @classmethod
    def GetNeighboursFromID(cls, id):
        return cls.m_id_neighbours.get(id, [])                

class StationNode:
    def __init__(self, node_id):
        self.node_id = node_id
        self.neighbours = []

    def InsertNeighbours(self, neighbour_ids):
        self.neighbours.extend(neighbour_ids)

    def GetNeighbours(self):
        return self.neighbours
    
class StationTree:
    def __init__(self, from_station_id):
        self.root = StationNode(from_station_id)

    def GetPathsTo(self, to_station_id):
        paths = []
        visited = set()  # Add a set to keep track of visited nodes
        queue = [(self.root, [self.root.node_id])]
        while queue:
            current_node, path = queue.pop(0)
            # Check here if the node has been visited already
            if current_node.node_id in visited:
                continue
            visited.add(current_node.node_id)  # Mark the current node as visited
            if current_node.node_id == to_station_id:
                paths.append(path)
                continue
            for neighbour_id in StationInfo.GetNeighboursFromID(current_node.node_id):
                if neighbour_id not in visited:  # Check if the neighbor is not visited
                    new_node = StationNode(neighbour_id)
                    queue.append((new_node, path + [neighbour_id]))
        return paths

class StationFinder:
    def __init__(self):
        self.station_info = StationInfo

    def GetPaths(self, from_station_name, to_station_name):
        from_station_id = self.station_info.GetIDFromName(from_station_name)
        to_station_id = self.station_info.GetIDFromName(to_station_name)
        if from_station_id == "Station name not found" or to_station_id == "Station name not found":
            return []
        tree = StationTree(from_station_id)
        return tree.GetPathsTo(to_station_id)

# This is the part where LoadData is called and other operations are performed
if __name__ == "__main__":
    StationInfo.LoadData('/Users/zhangxiyun/LondonPath/stations.txt')  # Make sure the file path is correct
    from_station_name = 'LiverpoolStreetStation'
    to_station_name = 'Whitechapel'

    # Create a StationFinder object
    finder = StationFinder()

    # Use the StationFinder object to find all paths from the start station to the destination station
    possible_paths = finder.GetPaths(from_station_name, to_station_name)

    # Check if any paths were found
    if possible_paths:
        # If paths were found, print them
        print(f"Found {len(possible_paths)} paths from {from_station_name} to {to_station_name}:")
        for i, path in enumerate(possible_paths, start=1):
            path_names = [StationInfo.GetNameFromID(station_id) for station_id in path]
            path_str = ' -> '.join(path_names)
            print(f"Path {i}: {path_str}")
    else:
        # If no paths were found, print an appropriate message
        print(f"No paths found from {from_station_name} to {to_station_name}")

```