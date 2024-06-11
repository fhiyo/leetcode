# 200. Number of Islands

## 1st

### ①

- 1 <= m, n <= 300 なので再帰の回数は高々90Kで、leetcodeの環境で `sys.getrecursionlimit()` で上限を調べてみると `550000` だったので耐えられはしそう。sys.setrecursionlimit()で増やしてもいいが。
  - だからといって再帰を選ぶかというと微妙かもしれない。スタックを使う方が無難かも
- gridを書き換えるかは迷ったが、今回は許される環境だとして実装した。
- `m = len(grid)`, `n = len(grid[0])` のように長さを別途変数にするかも迷ったが、2次元配列のgridが長方形を表していることが分かりやすいかと思いこうした。
- gridをinner function内で触る際、nonlocalにしなくていいんだっけ？と気になったが、grid自体をrebindするわけではないので問題ない。
- WATERの変数を使ってないことは気になる。無いのもまた違和感あるので、コメントにしてもいいかもしれない
- WATERなどのクラス変数をEnumにするか考えたが、Enumは遅いので使うのを見送った。Enumを使うメリットが速度 (などの) デメリットを上回るかは注意する必要がある

- https://stackoverflow.com/questions/27735732/why-are-python-enums-slow

    実験すると5倍くらいは遅かった。
    ```py
    >>> import timeit
    >>> from enum import Enum, auto
    >>> class Foo(Enum):
    ...     hoge = auto()
    ...
    >>> class Bar:
    ...     hoge = 10
    ...
    >>> timeit.timeit(lambda : Foo.hoge.value)
    0.2203257498331368
    >>> timeit.timeit(lambda : Bar.hoge)
    0.050958292093127966
    ```


所要時間: 8:39
m: len(grid), n: len(grid[0])
- 時間計算量: O(mn)
- 空間計算量: O(mn)

```py
class Solution:
    WATER = '0'
    UNVISITED = '1'
    VISITED = '2'

    def numIslands(self, grid: List[List[str]]) -> int:
        # precondition: len(grid) > 0, len(grid[i]) == len(grid[j]) (i != j)
        m = len(grid)
        n = len(grid[0])

        def visit(i: int, j: int):
            if not (0 <= i < m and 0 <= j < n and grid[i][j] == Solution.UNVISITED):
                return
            grid[i][j] = Solution.VISITED
            visit(i-1, j)
            visit(i+1, j)
            visit(i, j-1)
            visit(i, j+1)

        num_islands = 0
        for i in range(m):
            for j in range(n):
                if grid[i][j] == Solution.UNVISITED:
                    visit(i, j)
                    num_islands += 1
        return num_islands
```

### ②

スタックを使う。

所要時間: 10:39 (`while not positions:` にしていたのを気づかずバグ取りで詰まる。)
m: len(grid), n: len(grid[0])
- 時間計算量: O(mn)
- 空間計算量: O(mn)

```py
class Solution:
    WATER = '0'
    UNVISITED = '1'
    VISITED = '2'

    def numIslands(self, grid: List[List[str]]) -> int:
        # precondition: len(grid) > 0, len(grid[i]) == len(grid[j]) (i != j)
        m = len(grid)
        n = len(grid[0])

        def visit(start_row: int, start_col: int):
            positions = []
            positions.append((start_row, start_col))
            while positions:
                row, col = positions.pop()
                if not (0 <= row < m and 0 <= col < n and grid[row][col] == Solution.UNVISITED):
                    continue
                grid[row][col] = Solution.VISITED
                positions.append((row-1, col))
                positions.append((row+1, col))
                positions.append((row, col-1))
                positions.append((row, col+1))

        num_islands = 0
        for i in range(m):
            for j in range(n):
                if grid[i][j] == Solution.UNVISITED:
                    visit(i, j)
                    num_islands += 1
        return num_islands
```

### ③

gridを書き換えないバージョン。

所要時間: 12:59
m: len(grid), n: len(grid[0])
- 時間計算量: O(mn)
- 空間計算量: O(mn)

```py
class Solution:
    WATER = '0'
    LAND = '1'

    def numIslands(self, grid: List[List[str]]) -> int:
        # precondition: len(grid) > 0, len(grid[i]) == len(grid[j]) (i != j)
        m = len(grid)
        n = len(grid[0])
        visited = [[False] * n for _ in range(m)]

        def visit(i: int, j: int):
            if not (0 <= i < m and 0 <= j < n) or grid[i][j] != Solution.LAND or visited[i][j]:
                return
            visited[i][j] = True
            visit(i-1, j)
            visit(i+1, j)
            visit(i, j-1)
            visit(i, j+1)

        num_islands = 0
        for i in range(m):
            for j in range(n):
                if not visited[i][j] and grid[i][j] == Solution.LAND:
                    visit(i, j)
                    num_islands += 1
        return num_islands
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1233993712883732541
    - inside_island()のような内部関数を別途作るのは考えたが、内部関数をあまり多く作ると関数が追いにくくなるかな？と思い止めた。小さい関数一つだし作って良かったかも。適当な粒度でprivate methodに切り出すなど、柔軟にしていきたい。
- https://github.com/rossy0213/leetcode/pull/8
- https://discord.com/channels/1084280443945353267/1200089668901937312/1211329160413323295
  > ただ、m, n はわりと主役級なので、width, height などの名前をつけてしまってもいいかもしれません。

  m, nは問題文に合わせてそうしたが、プログラム単体で見たときにはそちらの方が分かりやすいのはそうだろう。プログラムのミスも減るだろうから、こちらを採用してみる。

- https://discord.com/channels/1084280443945353267/1183683738635346001/1194329732544737330
- https://discord.com/channels/1084280443945353267/1196472827457589338/1196541121912909824 の37

Union-Findでもできるらしいのでやってみた。全体的に出来が悪い気がするが、パッと改善案が思いつかない。

```py
from enum import Enum

class Terrain(Enum):
    WATER = '0'
    LAND = '1'


class Node:
    id_: int
    terrain: Terrain
    parent: 'Node'
    rank: int

    def __init__(self, id_: int, terrain: Terrain):
        self.id_ = id_
        self.terrain = Terrain(terrain)
        self.parent = self
        self.rank = 0


class DisjointSetForest:
    def __init__(self, grid: list[list[str]]):
        height = len(grid)
        width = len(grid[0])
        self.nodes = [Node(i * width + j, grid[i][j]) for i in range(height) for j in range(width)]

    def getLandRepresentatives(self):
        return list(filter(lambda n: n.terrain == Terrain.LAND and n == n.parent, self.nodes))

    def union(self, x_id: int, y_id: int):
        return self._link(self.findSet(x_id), self.findSet(y_id))

    def findSet(self, x_id: int):
        if self.nodes[x_id] != self.nodes[x_id].parent:
            self.nodes[x_id].parent = self.findSet(self.nodes[x_id].parent.id_)
        return self.nodes[x_id].parent

    def _link(self, x: Node, y: Node):
        if x.rank < y.rank:
            x.parent = y
        else:
            y.parent = x
            if x.rank == y.rank:
                x.rank += 1


class Solution:
    def numIslands(self, grid: List[List[str]]) -> int:
        # precondition: len(grid) > 0, len(grid[i]) == len(grid[j]) (i != j)
        height = len(grid)
        width = len(grid[0])

        def inner_grid(row: int, col: int) -> bool:
            return 0 <= row < height and 0 <= col < width

        sections = DisjointSetForest(grid)
        directions = [(-1, 0), (1, 0), (0, -1), (0, 1)]
        for row in range(height):
            for col in range(width):
                if grid[row][col] == Terrain.LAND.value:
                    for d_row, d_col in directions:
                        if inner_grid(row + d_row, col + d_col) and grid[row + d_row][col + d_col] == Terrain.LAND.value:
                            sections.union(row * width + col, (row + d_row) * width + (col + d_col))
        return len(sections.getLandRepresentatives())
```


## 3rd

```py
class Solution:
    # WATER = '0'
    UNVISITED = '1'
    VISITED = '2'

    def numIslands(self, grid: List[List[str]]) -> int:
        # precondition: len(grid) > 0, len(grid[i]) == len(grid[j]) (i != j)
        height = len(grid)
        width = len(grid[0])

        def inner_grid(row: int, col: int) -> bool:
            return 0 <= row < height and 0 <= col < width

        def visit(start_row: int, start_col: int):
            positions = [(start_row, start_col)]
            while positions:
                row, col = positions.pop()
                if not inner_grid(row, col) or grid[row][col] != Solution.UNVISITED:
                    continue
                grid[row][col] = Solution.VISITED
                positions.append((row - 1, col))
                positions.append((row + 1, col))
                positions.append((row, col - 1))
                positions.append((row, col + 1))

        num_islands = 0
        for i in range(height):
            for j in range(width):
                if grid[i][j] == Solution.UNVISITED:
                    visit(i, j)
                    num_islands += 1
        return num_islands
```
