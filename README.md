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

### Q7

Zvolil jsem heuristickou funkci, která hodnotí podle nejvzdálenějšího jídla od dané pozice. Funkce zkontroluje, jesli v bludišti jsou ještě nějaké "tečky" jídla. Pokud ano, pak přes ně iteruje a pro každou pozici spočítá její vzdálenost od Pacmana (důležité je, že používáme mazeDistance - tedy výpočet respektuje zdi bludiště - hodnota je tak přesnější, než při použití Manhattan distance). Vypočtené vzdálenosti se ukládají do `problem.heuristicInfo`, aby se omezilo opakovaným výpočtům stejných vzdáleností. Myšlenka je, že Pacman musí postupně navštívit všechny pozice jídla, tudíž musí urazit aspoň takovou vzdálenost, která odpovídá nejvzdálenější pozici jídla od současné pozice. Obecně tedy platí, že funkce vrací vzdálenost ze současné k nejvzdálenějšímu jídlu.

```python
def foodHeuristic(state: Tuple[Tuple, List[List]], problem: FoodSearchProblem):
    position, food_grid = state
    food_positions = food_grid.asList()

    if not food_positions:
        return 0

    distances = []
    for food_position in food_positions:
        key = (position, food_position)
        if key not in problem.heuristicInfo:
            problem.heuristicInfo[key] = mazeDistance(position, food_position, problem.startingGameState)
        distances.append(problem.heuristicInfo[key])

    return max(distances)
```

### Q8

Bylo potřeba doplnit funkci `ClosestDotSearchAgent.findPathToClosestDot`. Funkce použije BFS pro nalezení nejbližšího jídla u kterého máme jistotu, že první nalezené jídlo je nejbližší (nebo jedno z nejbližších).

```python
class ClosestDotSearchAgent(SearchAgent):
    ...
    def findPathToClosestDot(self, gameState: pacman.GameState):
        problem = AnyFoodSearchProblem(gameState)
        return search.breadthFirstSearch(problem)
```

Dále bylo potřeba doplnit podmínku pro cílový stav ve funkci `AnyFoodSearchProblem.isGoalState`. Ta je jednoduchá, protože nám jde pouze o to, jestli se na dané pozici nachází jídlo. A protože stav je tvořen souřadnicemi `x` a `y`, můžeme lehce zkontrolovat jestli je na pozici `x,y` jídlo (`self.food[x][y] == True`).

```python
class AnyFoodSearchProblem(PositionSearchProblem):
    ...
    def isGoalState(self, state: Tuple[int, int]):
        x, y = state
        return self.food[x][y]
```
