# Pathfinding Visualizer

## Read Description File First: [Description](https://github.com/mperea9/PathfindingVisualizer/blob/main/Description-Of-Algorithm.md)

## Relevant video: https://www.youtube.com/watch?v=EBlNqnWcTHA&list=LL&index=15

## Source Code: https://github.com/mperea9/PathfindingVisualizer/blob/main/PathVis.py

# My Code with Comments:

## To Start (Imports, Display size, Defining colors)

```
import pygame
import math
from queue import PriorityQueue

WIDTH = 800                                         # defining width of window, to make it square we only use a width
WIN = pygame.display.set_mode((WIDTH, WIDTH))       # setting up how big the display will be WIDTH, WIDTH will make it square as it is WIDTH x WIDTH
pygame.display.set_caption("A Star Algorithm")      # this is a caption header to our windowed algorithm

RED = (255, 0, 0)                                   # define all the colors for our different objects (start, end, path, nodes, barriers, etc.)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
YELLOW = (255, 255, 0)
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
PURPLE = (128, 0, 128)
ORANGE= (255, 165, 0)
GREY = (128, 128, 128)
TURQUOISE = (64, 224, 208)
```

## Setting our Node Class with methods

```
class Node:                                             # define class as Node to create our main visualizer tool before our algorithm, tracks all our nodes
    def __init__(self, row, col, width, total_rows):    
        self.row = row
        self.col = col
        self.x = row * width                            # keep track of x and y cooridinates on the screen
        self.y = col * width
        self.color = WHITE                              # start with all white cubes
        self.neighbors = []                             
        self.width = width
        self.total_rows = total_rows
        
    def get_pos(self):                                  # indexing with row and columns
        return self.row, self.col
                                                        # these methods just ask what node is, so if RED then it is closed, etc.
    def is_closed(self):                                # have we already looked at the node (in closed set), which is defined by red color
        return self.color == RED
    
    def is_open(self):                                  # are you in open set, set by color green
        return self.color == GREEN
    
    def is_barrier(self):                               # if it is a barrier, set to black
        return self.color == BLACK
    
    def is_start(self):                                 # if node is the start node, then set to orange
        return self.color == ORANGE
    
    def is_end(self):                                   # if node is the end node, then set to turquoise
        return self.color == TURQUOISE
    
    def reset(self):                                    # reset the node, set to white
        self.color = WHITE
        
    def make_closed(self):                              # these methods make the node and change the color, closed set to RED and same idea with the methods
        self.color = RED
    
    def make_open(self):
        self.color = GREEN
    
    def make_barrier(self):
        self.color = BLACK
    
    def make_start(self):
        self.color = ORANGE
    
    def make_end(self):
        self.color = TURQUOISE
    
    def make_path(self):                                            # allows us to make a path which is set to purple
        self.color = PURPLE
    
    def draw(self, win):                                                                    # method we call when we want to draw the node on the screen
        pygame.draw.rect(win, self.color, (self.x, self.y, self.width, self.width))         # window, color, x, y, width, height
        
    def update_neighbors(self, grid):                       # checks neighbors, if they aren't a barrier then the node is added into neighbors list
        self.neighbors = []
        if self.row < self.total_rows - 1 and not grid[self.row + 1][self.col].is_barrier(): # checks node that is directly down to see if it is a barrier
            self.neighbors.append(grid[self.row + 1][self.col])     # appends node into neighbors list

        if self.row > 0 and not grid[self.row - 1][self.col].is_barrier(): # checks node that is directly up to see if it is a barrier
            self.neighbors.append(grid[self.row - 1][self.col])     # appends node into neighbors list

        if self.col < self.total_rows - 1 and not grid[self.row][self.col + 1].is_barrier(): # checks node that is to the right to see if it is a barrier
            self.neighbors.append(grid[self.row][self.col + 1])     # appends node into neighbors list

        if self.col > 0 and not grid[self.row][self.col - 1].is_barrier(): # checks node that is to the left to see if it is a barier
            self.neighbors.append(grid[self.row][self.col - 1])     # appends node into neighbors list
    
    def __lt__(self, other): 
        return False
```

## Seting up our functions

```
def h(p1, p2):                  # define heuristic function, which is the Manhattan distance (up, down, left, right)
    x1, y1 = p1
    x2, y2 = p2
    return abs(x1 - x2) + abs(y1 - y2)

def reconstruct_path(came_from, current, draw):    # reconstructs the shortest path found
    while current in came_from:                     # while current is in came from table
        current = came_from[current]                
        current.make_path()                         # creates the path through each node
        draw()                                      # current node is end node and reconstructs to the start node from each came_from node

def algorithm(draw, grid, start, end):                 
    count = 0                                   # count is set to 0
    open_set = PriorityQueue()                  # create an open set
    open_set .put((0, count, start))            # add start node to priority queue
    came_from = {}                              # keeping track of where each node came from, keeps track of path
    g_score = {node: float("inf") for row in grid for node in row}      # create table for each g score
    g_score[start] = 0
    f_score = {node: float("inf") for row in grid for node in row}      # create table for each f score
    f_score[start] = h(start.get_pos(), end.get_pos())                  # f score set to heuristic, f score is set to distance from start to end node
    
    open_set_hash = {start}             # checks items are in or out of priority queue
    
    while not open_set.empty():             # algorithm runs until open set is empty
        for event in pygame.event.get():
            if event.type == pygame.QUIT:       # allows us to exit while loop
                pygame.quit()
                
        current = open_set.get()[2]             # current node popped out of queue
        open_set_hash.remove(current)           # take node we popped and sync it with open set hash to make sure we don't have duplicates
        
        if current == end: #make path                   # if node we popped is end node then we are finished and found a path
            reconstruct_path(came_from, end, draw)      # reconstructs the shortest path found
            end.make_end()
            return True
        
        for neighbor in current.neighbors:              # consider all neighbors of current node
            temp_g_score = g_score[current] + 1         # take distance from node and add 1 because it is neighbor
            
            if temp_g_score < g_score[neighbor]:        # if we found better path then we update path and store it to determine best path to each node
                 came_from[neighbor] = current          # updating the came from node to current
                 g_score[neighbor] = temp_g_score
                 f_score[neighbor] = temp_g_score + h(neighbor.get_pos(), end.get_pos())       
                 if neighbor not in open_set_hash:              # check if neighbor is in open set we then increment it
                     count += 1
                     open_set.put((f_score[neighbor], count, neighbor))     # put in this new neighbor since it has better path
                     open_set_hash.add(neighbor)                            # store the node in open set
                     neighbor.make_open()                                   
                     
        draw()  
        
        if current != start:            # if current is not the start node then close it, we can make the node red since it won't be reconsidered
            current.make_closed()
            
    return False
                 

def make_grid(rows, width):             # data structure that can hold all nodes to manipulate grid, how many rows and what width you want
    grid = []                           # grid is set to blank lise
    gap = width // rows                 # using integer divison to get width of each node should be
    for i in range(rows):               # in grid row i append node into it so we have lists inside lists that each store nodes
        grid.append([])
        for j in range(rows):
            node = Node(i, j, gap, rows)
            grid[i].append(node)
            
    return grid                         # grid is created


def draw_grid(win, rows, width):        # function to draw grid lines
    gap = width // rows
    for i in range(rows):
        pygame.draw.line(win, GREY, (0, i * gap), (width, i * gap))         # for every row we draw a horizontal line
        for j in range(rows):
            pygame.draw.line(win, GREY, (j * gap, 0), (j * gap, width))     # flip coordinates, so for every column we draw a vertical line
            

def draw(win, grid, rows, width):           # make function that draws everything
    win.fill(WHITE)                         # fills entire screen with white and redraw everything per frame
    
    for row in grid:
        for node in row:
            node.draw(win)
            
    draw_grid(win, rows, width)            # draw grid lines on top
    pygame.display.update()
    
    
def get_clicked_pos(pos, rows, width):      # takes mouse position and what node we actually clicked on
    gap = width // rows
    y, x = pos
    
    row = y // gap                          # we take position in x and y and dividing it by width of node to show where we clicked
    col = x // gap
    
    return row, col
```

## Setting up main function

```
def main(win, width):
    ROWS = 50                               # define amount of rows
    grid = make_grid(ROWS, width)           # generates the grid
    
    start = None                            # define the start and end position as none because we decide where
    end = None
    
    run = True                              # keep track of start and end position, if we started algorithm or not
    while run:
        draw(win, grid, ROWS, width)            # draws grid every loop
        for event in pygame.event.get():      # loops through all events to check them
            if event.type == pygame.QUIT:       # if we press exit then pygame window quits
                run = False
            
            if pygame.mouse.get_pressed()[0]:       # check mouse button pressed, left mouse button = 0
                pos = pygame.mouse.get_pos()                        # gives us position of mouse on screen
                row, col = get_clicked_pos(pos, ROWS, width)        # give us row and col we clicked on, specific node
                node = grid[row][col]                               # index node in grid
                if not start and node != end:                       
                    start = node                                    # makes start node, first thing you should create
                    start.make_start()
                    
                elif not end and node != start:
                    end = node                                      # makes end node, second thing you should create
                    end.make_end()
                    
                elif node != end and node != start:                 # if node is not end or start node, then it is a barrier
                    node.make_barrier()
            
            elif pygame.mouse.get_pressed()[2]:           # check mouse button pressed, right mouse button = 2
                pos = pygame.mouse.get_pos()                # gives us position of mouse on screen
                row, col = get_clicked_pos(pos, ROWS, width)    # gives us row and col we pressed on, specific node
                node = grid[row][col]                       # index node in grid
                node.reset()                                # this resets node to white
                if node == start:                           # if we click on start node, then reset start node and make it next to be made
                    start = None
                elif node == end:                           # if we click on end node, then reset end node and make it next to be made
                    end = None
                    
            if event.type == pygame.KEYDOWN:                # if we pressed a key on our keyboard
                if event.key == pygame.K_SPACE and start and end:       # if we pressed space then we run the algorithm as long as we have start and end node
                    for row in grid:
                        for node in row:
                            node.update_neighbors(grid)         # call method to update neighbors and check
                    
                    algorithm(lambda: draw(win, grid, ROWS, width), grid, start, end)       # allows us to pass draw function when we run algorithm method
                    
                if event.key == pygame.K_c:             # if we press the C key on our keyboard then it clears the board
                    start = None                        # sets start and end node to none as we need to reset them
                    end = None
                    grid = make_grid(ROWS, width)       # cleaes and redraws the grid
                                    
    pygame.quit()
    
main(WIN, WIDTH)
```
