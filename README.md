## Project: 3D Motion Planning
![Quad Image](./misc/enroute.png)

---
## How to run
```
 python motion_planning.py --goal_lon -122.40195876 --goal_lat 37.79673913 --goal_alt -0.147
 ```

## [Rubric](https://review.udacity.com/#!/rubrics/1534/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Explain the Starter Code

#### 1. Explain the functionality of what's provided in `motion_planning.py` and `planning_utils.py`
These scripts contain a basic planning implementation that includes...

Here is a video of my result.
https://youtu.be/oQS_rbUXzY8

`motion_planing.py` and 'planning_utils.py' includes a state based control loop for managing the drones behaviour, and planning a suitable path. The functionality in planning_utils.py discretizes the world into a grid or graph representation, while the motion_planning.py code implements the drone object for controlling the behaviour and mission of the drone.

An overview of the two solutions provided are as follows:

Feature | backyard_flyer | motion_planning
--- | --- | ---
`Implements a path planner for finding paths` | no | **yes**
`Uses dynamic searching for missions` | no | **yes**
`Includes an internal state of the configuration space` | no | **yes**
`Uses global positioning for following paths` | no | **yes**

### Implementing Your Path Planning Algorithm

#### 1. Set your global home position
The global home location is the first line of the csv file `colliders.csv`. The `z` component is zero at the beginning.

```
lat0, lon0 = read_home('./colliders.csv')
```

The extraction of lat0 and lon0 is implemented as a utility method `read_home` in `utils.py`. Then I use the self.set_home_position() method to set global home.

```
def read_home(filename):
    with open(filename, 'r') as f:
        line = f.readline()
        tmp = line.split(', ')
        lat0 = float(tmp[0].split(' ')[1])
        lon0 = float(tmp[1].split(' ')[1])
        return lat0, lon0
```

#### 2. Set your current local position

```
self.set_home_position(lon0, lat0, 0)
```

#### 3. Set grid start position from local position
The world is discretized. Here we set the bound for each dimension such that North and East have minimum bounds and maximum bounds. This is to reduce the entire grid size.

```
# Read in obstacle map
data = np.loadtxt('colliders.csv', delimiter=',', dtype='Float64', skiprows=2)

# Define a grid for a particular altitude and safety margin around obstacles
grid, north_offset, east_offset = create_grid(data, TARGET_ALTITUDE, SAFETY_DISTANCE)
```

The offset calculation is implemented in `create_grid` [`planning_utils.py`](planning_utils.py#L6)

Then the start grid point is implemented as the following code:

```
grid_start_north = int(np.ceil(north - north_offset))
grid_start_east = int(np.ceil(east - east_offset))
grid_start = (grid_start_north, grid_start_east)
```

#### 4. Set grid goal position from geodetic coords

```
north, east, down = global_to_local(self.global_position, self.global_home)
```

#### 5. Modify A* to include diagonal motion (or replace A* altogether)
The path search algorithm is replaced with the A* algorithm. The code is implemented in `a_star` [`planning_utils.py`](planning_utils.py#L91)

#### 6. Cull waypoints 
The path is pruned using collinearity as described in the lecture. If three points are detected to be collinear, then the middle point is removed. The code is implemented in `collinearity_prune` [`utils.py`](utils.py#L21)


### Execute the flight
#### 1. Does it work?
It works!
