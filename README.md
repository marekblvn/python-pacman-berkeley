### Q1

```python
def depthFirstSearch(problem: SearchProblem) -> List[Directions]:
    frontier = util.Stack()
    frontier.push((problem.getStartState(), []))
    visited = set()

    while not frontier.isEmpty():
        state, path = frontier.pop()

        if state in visited:
            continue
        visited.add(state)

        if problem.isGoalState(state):
            return path

        for successor, action, _ in problem.getSuccessors(state):
            if successor not in visited:
                frontier.push((successor, path + [action]))

    return []
```

### Q2

```python
def breadthFirstSearch(problem: SearchProblem) -> List[Directions]:
    frontier = util.Queue()
    frontier.push((problem.getStartState(), []))
    visited = set()
    while not frontier.isEmpty():
        state, path = frontier.pop()
        if state in visited:
            continue
        visited.add(state)

        if problem.isGoalState(state):
            return path

        for successor, action, _ in problem.getSuccessors(state):
            if successor not in visited:
                frontier.push((successor, path + [action]))
    return []
```

### Q3

```python
def uniformCostSearch(problem: SearchProblem) -> List[Directions]:
    frontier = util.PriorityQueue()
    frontier.push((problem.getStartState(), [], 0), 0)
    visited = set()

    while not frontier.isEmpty():
        state, path, cost = frontier.pop()

        if state in visited:
            continue
        visited.add(state)

        if problem.isGoalState(state):
            return path

        for successor, action, step_cost in problem.getSuccessors(state):
            if successor not in visited:
                next_cost = cost + step_cost
                frontier.push((successor, path + [action], next_cost), next_cost)
    return []
```
