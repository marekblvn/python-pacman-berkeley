# Semestrální práce - 4IZ431 - Umělá inteligence 1

## Q1: Finding a Fixed Food Dot using Depth First Search

Používáme zásobník (stack / LIFO), takže nově přidaný stav je hned expandován.

```python
def depthFirstSearch(problem: SearchProblem) -> List[Directions]:
    fringe = util.Stack()
    fringe.push((problem.getStartState(), []))
    visited = set()

    while not fringe.isEmpty():
        state, path = fringe.pop()

        if state in visited:
            continue
        visited.add(state)

        if problem.isGoalState(state):
            return path

        for successor, action, _ in problem.getSuccessors(state):
            if successor not in visited:
                fringe.push((successor, path + [action]))

    return []
```

## Q2: Breadth First Search

Používáme frontu (queue / FIFO), takže expandujeme stavy v pořadí v jakém jsme je přidali do fringe (navštívili).

```python
def breadthFirstSearch(problem: SearchProblem) -> List[Directions]:
    fringe = util.Queue()
    fringe.push((problem.getStartState(), []))
    visited = set()
    while not fringe.isEmpty():
        state, path = fringe.pop()
        if state in visited:
            continue
        visited.add(state)

        if problem.isGoalState(state):
            return path

        for successor, action, _ in problem.getSuccessors(state):
            if successor not in visited:
                fringe.push((successor, path + [action]))
    return []
```

## Q3: Varying the Cost Function

Používáme PriorityQueue, která při zavolání metody `pop` vrací prvek s nejnižší prioritou, takže UCS vždy expanduje nejprve nejlevnější cestu.

```python
def uniformCostSearch(problem: SearchProblem) -> List[Directions]:
    fringe = util.PriorityQueue()
    fringe.push((problem.getStartState(), [], 0), 0)
    visited = set()

    while not fringe.isEmpty():
        state, path, cost = fringe.pop()

        if state in visited:
            continue
        visited.add(state)

        if problem.isGoalState(state):
            return path

        for successor, action, step_cost in problem.getSuccessors(state):
            if successor not in visited:
                next_cost = cost + step_cost
                fringe.push((successor, path + [action], next_cost), next_cost)
    return []
```

## Q4: A\* Search

Implementoval jsem klasický A\* algoritmus - nevím co více k tomu napsat. Slovník `best_cost` slouží k uchovávání informace o nejmenší ceně do daného stavu. Slovník `expanded_cost` slouží k uchování informace o ceně již expandovaných stavů - ukládají cenu, kterou stav měl při expandování.

```python
def aStarSearch(problem: SearchProblem, heuristic=nullHeuristic) -> List[Directions]:
    fringe = util.PriorityQueue()
    start = problem.getStartState()
    fringe.push((start, [], 0), heuristic(start, problem))
    best_cost = {start: 0}
    expanded_cost = {}

    while not fringe.isEmpty():
        state, path, cost = fringe.pop()

        if state in expanded_cost and cost > expanded_cost[state]:
            continue
        expanded_cost[state] = cost

        if problem.isGoalState(state):
            return path

        for successor, action, step_cost in problem.getSuccessors(state):
            next_cost = cost + step_cost
            if successor not in best_cost or next_cost < best_cost[successor]:
                best_cost[successor] = next_cost
                priority = next_cost + heuristic(successor, problem)
                fringe.push((successor, path + [action], next_cost), priority)

    return []
```

## Q5: Finding All the Corners

```python
class CornersProblem(search.SearchProblem):
    def getStartState(self):
        visited_corners = frozenset(
            corner for corner in self.corners
            if corner == self.startingPosition
        )
        return (self.startingPosition, visited_corners)
```

Počáteční stav je tuple současné pozice a počtu navštívených rohů. Pokud Pacman začíná v rohu, máme ošetřené, že daný roh počítáme jako navštívený.

```python
    def isGoalState(self, state: Any):
        _, visited_corners = state
        return len(visited_corners) == len(self.corners)
```

Cílový stav je stav ve kterém jsou navštíveny všechny rohy.

```python
    def getSuccessors(self, state: Any):
        successors = []
        for action in [Directions.NORTH, Directions.SOUTH, Directions.EAST, Directions.WEST]:

            position, visited_corners = state
            x, y = position
            dx, dy = Actions.directionToVector(action)
            nextx, nexty = int(x + dx), int(y + dy)

            if not self.walls[nextx][nexty]:
                next_position = (nextx, nexty)
                next_visited_corners = visited_corners
                if next_position in self.corners:
                    next_visited_corners = visited_corners | frozenset([next_position])
                successors.append(((next_position, next_visited_corners), action, 1))

        self._expanded += 1 # DO NOT CHANGE
        return successors
```

Pro každý možný směr vypočítáme budoucí souřadnice. Pokud na dané souřadnici není zeď, vytvoříme potomka, který obsahuje budoucí stav (budoucí pozice a budoucí navštívené rohy), akci (tzn. směr kterým se Pacman pohnul) a cenu - ta je vždy =1. Je-li budoucí pozice v rohu, pak ten roh označíme jako navštívený.

## Q6: Corners Problem: Heuristic

Heuristická funkce která odhaduje kolik kroků je potřeba k dosažení nenavštívených rohů bludiště. Funkce najde nejbližší nenavštívený roh pomocí Manhattan vzdálenosti a jeho vzdálenost přičte k celkové heuristické hodnotě a takto dokud nejsou navštíveny všechny rohy bludiště. Jinými slovy - ze současné pozice Pacmana jdeme do nejbližšího nenavštíveného rohu dokud nějaký takový existuje. Následně sečteme uražené vzdálenosti.

```python
def cornersHeuristic(state: Any, problem: CornersProblem):
    position, visited_corners = state
    unvisited_corners = set(problem.corners) - set(visited_corners)
    heuristic = 0
    current_position = position

    while unvisited_corners:
        nearest_corner = min(
            unvisited_corners,
            key=lambda corner: util.manhattanDistance(current_position, corner)
        )
        heuristic += util.manhattanDistance(current_position, nearest_corner)
        current_position = nearest_corner
        unvisited_corners.remove(nearest_corner)

    return heuristic
```

## Q7: Eating All the Dots

Zvolil jsem heuristickou funkci, která hodnotí podle nejvzdálenějšího jídla od dané pozice. Funkce zkontroluje, jesli v bludišti jsou ještě nějaké "tečky" jídla. Pokud ano, pak přes ně iteruje a pro každou pozici spočítá její vzdálenost od Pacmana (důležité je, že používáme mazeDistance - tedy výpočet respektuje zdi bludiště - hodnota je tak přesnější, než při použití Manhattan distance). Vypočtené vzdálenosti se ukládají do `problem.heuristicInfo`, aby se omezilo opakovaným výpočtům stejných vzdáleností. Myšlenka je, že Pacman musí postupně navštívit všechny pozice jídla, tudíž musí urazit aspoň takovou vzdálenost, která odpovídá nejvzdálenější pozici jídla od současné pozice. Obecně tedy platí, že funkce vrací vzdálenost ze současné pozice k nejvzdálenějšímu jídlu.

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

## Q8: Suboptimal Search

Bylo potřeba doplnit metodu `ClosestDotSearchAgent.findPathToClosestDot`. Funkce použije BFS pro nalezení nejbližšího jídla (případně jednoho z nejbližších jídel).

```python
class ClosestDotSearchAgent(SearchAgent):
    ...
    def findPathToClosestDot(self, gameState: pacman.GameState):
        problem = AnyFoodSearchProblem(gameState)
        return search.breadthFirstSearch(problem)
```

Dále bylo potřeba doplnit podmínku pro cílový stav v metodě `AnyFoodSearchProblem.isGoalState`. Ta je jednoduchá, protože nám jde pouze o to, jestli se na dané pozici nachází jídlo. A protože stav je tvořen souřadnicemi `x` a `y`, můžeme lehce zkontrolovat jestli je na pozici `x,y` jídlo (`self.food[x][y] == True`).

```python
class AnyFoodSearchProblem(PositionSearchProblem):
    ...
    def isGoalState(self, state: Tuple[int, int]):
        x, y = state
        return self.food[x][y]
```
