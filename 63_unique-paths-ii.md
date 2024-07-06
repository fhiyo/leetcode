# 63. Unique Paths II

## 1st

### ①

pathsという変数名は、num_pathsにするとnum_rows, num_colsと似てるけど別物になるのが嫌だったので変えた。変だと思うんだが良い変数名が出てこない。num_paths_per_colとか...？

1行目とそれ以外を区別して処理してしまったが、その必要はない。

obstacleGridの0, 1は定数にした方が良かった。

所要時間: 13:16

gridのサイズ: m x n
- 時間計算量: O(mn)
- 空間計算量: O(n)

```py
class Solution:
    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        # precondition: len(obstacleGrid) > 0, len(obstacleGrid[0]) > 0, len(obstacleGrid[i]) == len(obstacleGrid[j]) forall i != j
        if obstacleGrid[0][0] == 1:
            return 0
        num_rows = len(obstacleGrid)
        num_cols = len(obstacleGrid[0])
        paths = [0] * num_cols
        paths[0] = 1
        for col in range(1, num_cols):
            if obstacleGrid[0][col] == 0:
                paths[col] += paths[col - 1]
        for row in range(1, num_rows):
            if obstacleGrid[row][0] == 1:
                paths[0] = 0
            for col in range(1, num_cols):
                if obstacleGrid[row][col] == 1:
                    paths[col] = 0
                else:
                    paths[col] += paths[col - 1]
        return paths[-1]
```

### ②

メモ化再帰。2次元で長方形であることを保証しようとすると、assertでやるなら `assert len(grid) > 0`, `assert len(grid[0]) > 0` とループで `assert len(grid[i]) == len(grid[0])` みたいなことをしないといけないはずだが、やりすぎな気がしてやっていない。しかし前提にして書いていいのかはいつも迷ってしまう。

所要時間: 2:58

gridのサイズ: m x n
- 時間計算量: O(mn)
- 空間計算量: O(mn)

```py
class Solution:
    OBSTACLE = 1

    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        num_rows = len(obstacleGrid)
        num_cols = len(obstacleGrid[0])

        @cache
        def count_paths_from(row: int, col: int) -> int:
            if obstacleGrid[row][col] == Solution.OBSTACLE:
                return 0
            if row == num_rows - 1 and col == num_cols - 1:
                return 1
            count = 0
            if row < num_rows - 1:
                count += count_paths_from(row + 1, col)
            if col < num_cols - 1:
                count += count_paths_from(row, col + 1)
            return count

        return count_paths_from(0, 0)
```

## 2nd

### 参考

- https://discord.com/channels/1084280443945353267/1233603535862628432/1257363629272072282
- https://discord.com/channels/1084280443945353267/1192736784354918470/1252607758604308532
  ①で困っていたpathsという名前。こちらではunique_paths_tableにしている。 自分のに合わせるとnum_paths_rowになるか。pathの数が入っている列、なら何が入っているか分かるかも。
- https://discord.com/channels/1084280443945353267/1227073733844406343/1244288660309020712

これが一番素直かもしれない。テーブルを作って上と左からpathの数を足していく。

```py
class Solution:
    OBSTACLE = 1

    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        assert len(obstacleGrid) > 0 and len(obstacleGrid[0]) > 0
        num_rows = len(obstacleGrid)
        num_cols = len(obstacleGrid[0])
        num_paths_table = [[0] * num_cols for _ in range(num_rows)]
        num_paths_table[0][0] = 1
        for row in range(num_rows):
            for col in range(num_cols):
                if obstacleGrid[row][col] == Solution.OBSTACLE:
                    num_paths_table[row][col] = 0
                    continue
                if row == 0 and col == 0:
                    continue
                num_paths = 0
                if row > 0:
                    num_paths += num_paths_table[row - 1][col]
                if col > 0:
                    num_paths += num_paths_table[row][col - 1]
                num_paths_table[row][col] = num_paths
        return num_paths_table[-1][-1]
```

- https://discord.com/channels/1084280443945353267/1225849404037009609/1236740338937368737


## 3rd

```py
class Solution:
    OBSTACLE = 1

    def uniquePathsWithObstacles(self, obstacleGrid: List[List[int]]) -> int:
        num_rows = len(obstacleGrid)
        num_cols = len(obstacleGrid[0])
        num_paths_row = [0] * num_cols
        num_paths_row[0] = 1
        for row in range(num_rows):
            for col in range(num_cols):
                if obstacleGrid[row][col] == Solution.OBSTACLE:
                    num_paths_row[col] = 0
                    continue
                if col == 0:
                    continue
                num_paths_row[col] += num_paths_row[col - 1]
        return num_paths_row[-1]
```
