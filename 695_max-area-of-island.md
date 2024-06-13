# 695. Max Area of Island

## 1st

### ①

スタックを使った解法。 https://github.com/fhiyo/leetcode/pull/20 と同様、gridは書き換える方針で行く。

所要時間: 10:25

m: len(grid), n: len(grid[0])
- 時間計算量: O(mn)
- 空間計算量: O(mn)

```py
class Solution:
    UNVISITED = 1
    VISITED = 2

    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        nrows = len(grid)
        ncols = len(grid[0])

        def calculate_area(start_row: int, start_col: int) -> int:
            inner_grid = lambda r, c: 0 <= r < nrows and 0 <= c < ncols
            positions = [(start_row, start_col)]
            area = 0
            while positions:
                row, col = positions.pop()
                if not inner_grid(row, col) or grid[row][col] != Solution.UNVISITED:
                    continue
                area += 1
                grid[row][col] = Solution.VISITED
                positions.append((row - 1, col))
                positions.append((row + 1, col))
                positions.append((row, col - 1))
                positions.append((row, col + 1))
            return area

        max_area = 0
        for r in range(nrows):
            for c in range(ncols):
                if grid[r][c] == Solution.UNVISITED:
                    max_area = max(max_area, calculate_area(r, c))
        return max_area
```

### ②

union by size のDisjoint-Set Forestで出来そうだったので書いてみた。

所要時間: 13:59

m: len(grid), n: len(grid[0])
- 時間計算量: O(mnα(mn)) (償却), α(x): アッカーマン関数の逆関数
  - 証明は追えていない & 合っているのか分からない...
- 空間計算量: O(mn)


```py
class UnionFind:
    def __init__(self, n):
        self.parent = [i for i in range(n)]
        self.size = [1] * n

    def getSize(self, i: int) -> int:
        return self.size[self.findSet(i)]

    def findSet(self, i: int) -> int:
        if i == self.parent[i]:
            return i
        self.parent[i] = self.findSet(self.parent[i])
        return self.parent[i]

    def union(self, i: int, j: int):
        self._link(self.findSet(i), self.findSet(j))

    def _link(self, i: int, j: int):
        if i == j:
            return
        if self.size[i] < self.size[j]:
            self.parent[i] = j
            self.size[j] += self.size[i]
        else:
            self.parent[j] = i
            self.size[i] += self.size[j]


class Solution:
    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        nrows = len(grid)
        ncols = len(grid[0])
        inner_grid = lambda r, c: 0 <= r < nrows and 0 <= c < ncols
        index = lambda r, c: r * ncols + c
        uf = UnionFind(nrows * ncols)
        max_area = 0
        for r in range(nrows):
            for c in range(ncols):
                if grid[r][c] != 1:
                    continue
                if inner_grid(r + 1, c) and grid[r + 1][c] == 1:
                    uf.union(index(r, c), index(r + 1, c))
                if inner_grid(r, c + 1) and grid[r][c + 1] == 1:
                    uf.union(index(r, c), index(r, c + 1))
                max_area = max(max_area, uf.getSize(index(r, c)))
        return max_area
```

### ③

再帰を使った方法。m, nが高々50なので最大2500回再帰しうることを考えると選択肢には入りづらい気がする。環境によって変わるとはいえPythonのデフォルトの再帰上限は1000回。再帰の上限を確認したり上限を変更したうえでならやるか。

所要時間: 4:47

m: len(grid), n: len(grid[0])
- 時間計算量: O(mn)
- 空間計算量: O(mn)

```py
class Solution:
    UNVISITED = 1
    VISITED = 2

    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        nrows = len(grid)
        ncols = len(grid[0])

        def calculate_area(row: int, col: int) -> int:
            inner_grid = lambda r, c: 0 <= r < nrows and 0 <= c < ncols
            if not inner_grid(row, col) or grid[row][col] != Solution.UNVISITED:
                return 0
            grid[row][col] = Solution.VISITED
            area = 1
            area += calculate_area(row - 1, col)
            area += calculate_area(row + 1, col)
            area += calculate_area(row, col - 1)
            area += calculate_area(row, col + 1)
            return area

        max_area = 0
        for r in range(nrows):
            for c in range(ncols):
                if grid[r][c] == Solution.UNVISITED:
                    max_area = max(max_area, calculate_area(r, c))
        return max_area
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1227073733844406343/1234813898293514271
- https://discord.com/channels/1084280443945353267/1192736784354918470/1224508766922215506
- https://discord.com/channels/1084280443945353267/1200089668901937312/1211642725594824704

BFSでやるのを忘れていたので実装。ついでにgridを書き換えないようにもやってみた。

```py
class Solution:
    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        nrows = len(grid)
        ncols = len(grid[0])
        visited = [[False] * ncols for _ in range(nrows)]

        def calculate_area(start_row: int, start_col: int) -> int:
            inner_grid = lambda r, c: 0 <= r < nrows and 0 <= c < ncols
            positions = deque([(start_row, start_col)])
            area = 0
            while positions:
                row, col = positions.popleft()
                if not inner_grid(row, col) or grid[row][col] != 1 or visited[row][col]:
                    continue
                area += 1
                visited[row][col] = True
                positions.append((row - 1, col))
                positions.append((row + 1, col))
                positions.append((row, col - 1))
                positions.append((row, col + 1))
            return area

        max_area = 0
        for r in range(nrows):
            for c in range(ncols):
                if grid[r][c] != 1 or visited[r][c]:
                    continue
                max_area = max(max_area, calculate_area(r, c))
        return max_area
```

## 3rd

ここのpositionsという変数名は自然なんだろうか。pointsとかの方が良かったり？


```py
class Solution:
    UNVISITED = 1
    VISITED = 2

    def maxAreaOfIsland(self, grid: List[List[int]]) -> int:
        nrows = len(grid)
        ncols = len(grid[0])

        def calculate_area(start_row: int, start_col: int) -> int:
            inner_grid = lambda r, c: 0 <= r < nrows and 0 <= c < ncols
            positions = [(start_row, start_col)]
            area = 0
            while positions:
                row, col = positions.pop()
                if not inner_grid(row, col) or grid[row][col] != Solution.UNVISITED:
                    continue
                grid[row][col] = Solution.VISITED
                area += 1
                positions.append((row - 1, col))
                positions.append((row + 1, col))
                positions.append((row, col - 1))
                positions.append((row, col + 1))
            return area

        max_area = 0
        for r in range(nrows):
            for c in range(ncols):
                if grid[r][c] == Solution.UNVISITED:
                    max_area = max(max_area, calculate_area(r, c))
        return max_area
```
