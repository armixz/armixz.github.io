---
layout: page
title: Multi-Algorithm AI Search Engine
description: Sophisticated 8-puzzle solver with four distinct search algorithms
img: assets/img/9.jpg
importance: 9
category: work
github: https://github.com/armixz/8-Puzzle-Solver
---

## Project Overview

Implemented a **sophisticated 8-puzzle solver** using **four distinct search algorithms** in **Fall 2021**, including Depth-First Search, Iterative Deepening Search, and A* search with Manhattan distance and misplaced tile heuristics. The system achieves **optimal solutions** with a command-line interface and file-based input processing for automated testing and performance comparison.

## Problem Domain Analysis

### 8-Puzzle Game Definition
- **State Space**: 9! = 362,880 possible configurations
- **Goal State**: Arranged tiles 1-8 with empty space in bottom-right
- **Valid Moves**: Up, Down, Left, Right (based on empty space position)
- **Optimal Solution**: Minimum number of moves to reach goal state
- **Complexity**: NP-complete problem requiring intelligent search strategies

### State Representation
```python
class PuzzleState:
    def __init__(self, board, empty_pos=None, parent=None, move=None, depth=0):
        self.board = board  # 3x3 numpy array
        self.empty_pos = empty_pos or self._find_empty_position()
        self.parent = parent
        self.move = move
        self.depth = depth
        self.hash_value = hash(tuple(board.flatten()))
    
    def _find_empty_position(self):
        """Find position of empty tile (represented as 0)"""
        pos = np.where(self.board == 0)
        return (pos[0][0], pos[1][0])
    
    def get_successors(self):
        """Generate all valid successor states"""
        successors = []
        row, col = self.empty_pos
        moves = [('UP', -1, 0), ('DOWN', 1, 0), ('LEFT', 0, -1), ('RIGHT', 0, 1)]
        
        for move_name, dr, dc in moves:
            new_row, new_col = row + dr, col + dc
            if 0 <= new_row < 3 and 0 <= new_col < 3:
                new_board = self.board.copy()
                # Swap empty space with adjacent tile
                new_board[row, col], new_board[new_row, new_col] = \
                    new_board[new_row, new_col], new_board[row, col]
                
                successor = PuzzleState(
                    new_board, (new_row, new_col), self, move_name, self.depth + 1
                )
                successors.append(successor)
        
        return successors
```

## Search Algorithm Implementation

### 1. Depth-First Search (DFS)
```python
class DepthFirstSearch:
    def __init__(self, max_depth=50):
        self.max_depth = max_depth
        self.visited = set()
        self.nodes_expanded = 0
    
    def search(self, initial_state, goal_state):
        self.visited.clear()
        self.nodes_expanded = 0
        
        stack = [initial_state]
        
        while stack:
            current = stack.pop()
            self.nodes_expanded += 1
            
            if self._is_goal(current, goal_state):
                return self._reconstruct_path(current)
            
            if current.depth >= self.max_depth:
                continue
                
            state_hash = current.hash_value
            if state_hash in self.visited:
                continue
            
            self.visited.add(state_hash)
            
            # Add successors to stack (reverse order for left-to-right processing)
            successors = current.get_successors()
            for successor in reversed(successors):
                if successor.hash_value not in self.visited:
                    stack.append(successor)
        
        return None  # No solution found
```

### 2. Iterative Deepening Search (IDS)
```python
class IterativeDeepeningSearch:
    def __init__(self, max_depth=50):
        self.max_depth = max_depth
        self.nodes_expanded = 0
    
    def search(self, initial_state, goal_state):
        self.nodes_expanded = 0
        
        for depth_limit in range(self.max_depth + 1):
            visited = set()
            result = self._depth_limited_search(
                initial_state, goal_state, depth_limit, visited
            )
            
            if result is not None:
                return result
        
        return None  # No solution found within depth limit
    
    def _depth_limited_search(self, state, goal_state, depth_limit, visited):
        self.nodes_expanded += 1
        
        if self._is_goal(state, goal_state):
            return self._reconstruct_path(state)
        
        if state.depth >= depth_limit:
            return None
        
        state_hash = state.hash_value
        if state_hash in visited:
            return None
        
        visited.add(state_hash)
        
        for successor in state.get_successors():
            result = self._depth_limited_search(
                successor, goal_state, depth_limit, visited
            )
            if result is not None:
                return result
        
        return None
```

### 3. A* Search with Manhattan Distance Heuristic
```python
class AStarManhattan:
    def __init__(self):
        self.nodes_expanded = 0
        self.goal_positions = self._compute_goal_positions()
    
    def _compute_goal_positions(self):
        """Precompute goal positions for each tile"""
        positions = {}
        goal_board = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 0]])
        
        for i in range(3):
            for j in range(3):
                if goal_board[i, j] != 0:
                    positions[goal_board[i, j]] = (i, j)
        
        return positions
    
    def _manhattan_distance(self, state):
        """Calculate Manhattan distance heuristic"""
        distance = 0
        
        for i in range(3):
            for j in range(3):
                tile = state.board[i, j]
                if tile != 0:  # Skip empty space
                    goal_i, goal_j = self.goal_positions[tile]
                    distance += abs(i - goal_i) + abs(j - goal_j)
        
        return distance
    
    def search(self, initial_state, goal_state):
        self.nodes_expanded = 0
        
        # Priority queue: (f_score, unique_id, state)
        open_set = [(self._manhattan_distance(initial_state), 0, initial_state)]
        closed_set = set()
        unique_id = 1
        
        while open_set:
            _, _, current = heapq.heappop(open_set)
            self.nodes_expanded += 1
            
            if self._is_goal(current, goal_state):
                return self._reconstruct_path(current)
            
            state_hash = current.hash_value
            if state_hash in closed_set:
                continue
            
            closed_set.add(state_hash)
            
            for successor in current.get_successors():
                successor_hash = successor.hash_value
                if successor_hash not in closed_set:
                    g_score = successor.depth
                    h_score = self._manhattan_distance(successor)
                    f_score = g_score + h_score
                    
                    heapq.heappush(open_set, (f_score, unique_id, successor))
                    unique_id += 1
        
        return None
```

### 4. A* Search with Misplaced Tiles Heuristic
```python
class AStarMisplacedTiles:
    def __init__(self):
        self.nodes_expanded = 0
    
    def _misplaced_tiles(self, state):
        """Count number of misplaced tiles (excluding empty space)"""
        goal_board = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 0]])
        misplaced = 0
        
        for i in range(3):
            for j in range(3):
                if state.board[i, j] != 0 and state.board[i, j] != goal_board[i, j]:
                    misplaced += 1
        
        return misplaced
    
    def search(self, initial_state, goal_state):
        # Similar implementation to A* Manhattan, but using misplaced tiles heuristic
        self.nodes_expanded = 0
        
        open_set = [(self._misplaced_tiles(initial_state), 0, initial_state)]
        closed_set = set()
        unique_id = 1
        
        while open_set:
            _, _, current = heapq.heappop(open_set)
            self.nodes_expanded += 1
            
            if self._is_goal(current, goal_state):
                return self._reconstruct_path(current)
            
            state_hash = current.hash_value
            if state_hash in closed_set:
                continue
            
            closed_set.add(state_hash)
            
            for successor in current.get_successors():
                successor_hash = successor.hash_value
                if successor_hash not in closed_set:
                    g_score = successor.depth
                    h_score = self._misplaced_tiles(successor)
                    f_score = g_score + h_score
                    
                    heapq.heappush(open_set, (f_score, unique_id, successor))
                    unique_id += 1
        
        return None
```

## Performance Analysis Framework

### Algorithm Comparison Metrics
```python
class PerformanceAnalyzer:
    def __init__(self):
        self.algorithms = {
            'DFS': DepthFirstSearch(),
            'IDS': IterativeDeepeningSearch(),
            'A*_Manhattan': AStarManhattan(),
            'A*_Misplaced': AStarMisplacedTiles()
        }
    
    def compare_algorithms(self, test_cases):
        results = {}
        
        for case_name, (initial_state, goal_state) in test_cases.items():
            results[case_name] = {}
            
            for alg_name, algorithm in self.algorithms.items():
                start_time = time.time()
                solution = algorithm.search(initial_state, goal_state)
                end_time = time.time()
                
                results[case_name][alg_name] = {
                    'solution_found': solution is not None,
                    'solution_length': len(solution) if solution else None,
                    'nodes_expanded': algorithm.nodes_expanded,
                    'execution_time': end_time - start_time,
                    'memory_usage': self._get_memory_usage()
                }
        
        return results
    
    def generate_report(self, results):
        """Generate comprehensive performance comparison report"""
        report = []
        report.append("8-Puzzle Solver Performance Analysis")
        report.append("=" * 50)
        
        for case_name, case_results in results.items():
            report.append(f"\nTest Case: {case_name}")
            report.append("-" * 30)
            
            for alg_name, metrics in case_results.items():
                report.append(f"{alg_name}:")
                report.append(f"  Solution Found: {metrics['solution_found']}")
                report.append(f"  Solution Length: {metrics['solution_length']}")
                report.append(f"  Nodes Expanded: {metrics['nodes_expanded']}")
                report.append(f"  Execution Time: {metrics['execution_time']:.4f}s")
                report.append(f"  Memory Usage: {metrics['memory_usage']:.2f}MB")
                report.append("")
        
        return "\n".join(report)
```

## Command-Line Interface

### Interactive Puzzle Solver
```python
class PuzzleSolverCLI:
    def __init__(self):
        self.analyzer = PerformanceAnalyzer()
    
    def run(self):
        """Main command-line interface loop"""
        print("8-Puzzle Solver - Multi-Algorithm Search Engine")
        print("=" * 50)
        
        while True:
            print("\nOptions:")
            print("1. Solve puzzle interactively")
            print("2. Load puzzle from file")
            print("3. Run algorithm comparison")
            print("4. Generate random puzzle")
            print("5. Exit")
            
            choice = input("\nEnter your choice (1-5): ").strip()
            
            if choice == '1':
                self._interactive_solve()
            elif choice == '2':
                self._solve_from_file()
            elif choice == '3':
                self._run_comparison()
            elif choice == '4':
                self._generate_random_puzzle()
            elif choice == '5':
                print("Goodbye!")
                break
            else:
                print("Invalid choice. Please try again.")
    
    def _interactive_solve(self):
        """Interactive puzzle input and solving"""
        print("\nEnter the puzzle state (3x3 grid, use 0 for empty space):")
        board = []
        
        for i in range(3):
            while True:
                try:
                    row = list(map(int, input(f"Row {i+1}: ").split()))
                    if len(row) != 3:
                        raise ValueError("Each row must have exactly 3 numbers")
                    board.append(row)
                    break
                except ValueError as e:
                    print(f"Invalid input: {e}. Please try again.")
        
        initial_state = PuzzleState(np.array(board))
        goal_state = PuzzleState(np.array([[1, 2, 3], [4, 5, 6], [7, 8, 0]]))
        
        # Check if puzzle is solvable
        if not self._is_solvable(initial_state):
            print("This puzzle configuration is not solvable!")
            return
        
        print("\nSelect algorithm:")
        algorithms = ['DFS', 'IDS', 'A*_Manhattan', 'A*_Misplaced']
        for i, alg in enumerate(algorithms, 1):
            print(f"{i}. {alg}")
        
        while True:
            try:
                alg_choice = int(input("Enter algorithm choice (1-4): ")) - 1
                if 0 <= alg_choice < len(algorithms):
                    break
                else:
                    print("Invalid choice. Please enter 1-4.")
            except ValueError:
                print("Please enter a valid number.")
        
        algorithm_name = algorithms[alg_choice]
        algorithm = self.analyzer.algorithms[algorithm_name]
        
        print(f"\nSolving puzzle using {algorithm_name}...")
        start_time = time.time()
        solution = algorithm.search(initial_state, goal_state)
        end_time = time.time()
        
        if solution:
            print(f"Solution found in {len(solution)} moves!")
            print(f"Moves: {' -> '.join(solution)}")
            print(f"Nodes expanded: {algorithm.nodes_expanded}")
            print(f"Execution time: {end_time - start_time:.4f} seconds")
        else:
            print("No solution found within the search limits.")
```

## File-Based Input Processing

### Automated Testing Framework
```python
def load_test_cases_from_file(filename):
    """Load puzzle test cases from file for batch processing"""
    test_cases = {}
    
    with open(filename, 'r') as file:
        current_case = None
        initial_board = []
        goal_board = []
        reading_initial = False
        reading_goal = False
        
        for line in file:
            line = line.strip()
            
            if line.startswith('CASE:'):
                current_case = line.split(':')[1].strip()
                initial_board = []
                goal_board = []
            elif line == 'INITIAL:':
                reading_initial = True
                reading_goal = False
            elif line == 'GOAL:':
                reading_initial = False
                reading_goal = True
            elif line == 'END':
                if current_case and initial_board and goal_board:
                    initial_state = PuzzleState(np.array(initial_board))
                    goal_state = PuzzleState(np.array(goal_board))
                    test_cases[current_case] = (initial_state, goal_state)
                reading_initial = False
                reading_goal = False
            elif reading_initial and line:
                row = list(map(int, line.split()))
                initial_board.append(row)
            elif reading_goal and line:
                row = list(map(int, line.split()))
                goal_board.append(row)
    
    return test_cases
```

## Key Achievements

### Optimality and Performance
- **Optimal Solutions**: A* algorithms guarantee shortest path to goal
- **Completeness**: All algorithms find solution if one exists (within limits)
- **Efficiency**: Manhattan distance heuristic outperforms misplaced tiles
- **Scalability**: Handles complex puzzle configurations efficiently

### Algorithm Comparison Results
- **A* Manhattan**: Best overall performance (fewest nodes expanded)
- **A* Misplaced Tiles**: Good performance, simpler heuristic calculation
- **Iterative Deepening**: Memory efficient, optimal solutions
- **Depth-First Search**: Fastest per node, but may find suboptimal solutions

## Technologies Used

- **Python** for algorithm implementation and framework development
- **NumPy** for efficient array operations and state representation
- **Heapq** for priority queue implementation in A* search
- **Time/Memory Profiling** for performance analysis
- **File I/O** for automated testing and batch processing

The project demonstrates expertise in artificial intelligence search algorithms, algorithm analysis, performance optimization, and software engineering practices essential for AI research, game development, and optimization problem solving.
