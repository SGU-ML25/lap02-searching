# Giải quyết vấn đề bằng tìm kiếm – Trả lời lý thuyết Bài 6, 7, 8

> Họ tên / MSSV: ……………………………………
>
> Notebook kèm theo:
> - Bài 6 → `06_Explore_heuristics.ipynb` (phần "Bài 6 – Ảnh hưởng của heuristic" ở cuối notebook)
> - Bài 7 → `05_Maze_Example-Animation_Cycle_checking.ipynb` (phần "Bài 7 – So sánh ba phương án cycle checking")
> - Bài 8 → `Ex1_Maze.ipynb` (bản rút gọn chỉ BFS: `Ex2_Maze_simplified.ipynb`)
>
> Mọi số liệu dưới đây lấy từ lần chạy các notebook trên. Thứ tự sinh hành động cố định là N, E, S, W nên kết quả tái lập được; chỉ có cột thời gian thay đổi giữa các lần chạy.

---

## Mục lục

1. [Ký hiệu chung](#1-ký-hiệu-chung) – kèm 1.1 Tổ chức mã nguồn và các thay đổi trong notebook
2. [Bài 6 – Ảnh hưởng của heuristic](#2-bài-6--ảnh-hưởng-của-heuristic) – mã nguồn ở 2.7
3. [Bài 7 – Kiểm tra chu trình trong DFS](#3-bài-7--kiểm-tra-chu-trình-trong-dfs) – mã nguồn ở 3.6
4. [Bài 8 – Goal-based Planning Agent](#4-bài-8--goal-based-planning-agent) – mã nguồn ở 4.10 (Ex1) và 4.11 (Ex2)
5. [Câu hỏi thảo luận (mục 4.3 của đề)](#5-câu-hỏi-thảo-luận-mục-43-của-đề)
6. [Tóm tắt](#6-tóm-tắt)

---

## 1. Ký hiệu chung

| Ký hiệu | Ý nghĩa |
|---|---|
| $n$ | số trạng thái (số ô không phải tường) |
| $b$ | hệ số phân nhánh tối đa (≤ 4 trong mê cung) |
| $d$ | độ sâu của lời giải nông nhất (= chi phí tối ưu, vì mỗi bước có chi phí 1) |
| $m$ | độ sâu tối đa của cây tìm kiếm |
| $g(n)$ | chi phí đường đi từ gốc tới nút $n$ |
| $h(n)$ | ước lượng chi phí từ $n$ tới đích; $h^*(n)$ là chi phí thật |
| $f(n)$ | hàm đánh giá dùng để chọn nút từ frontier |
| $C^*$ | chi phí của lời giải tối ưu |

Hàm đánh giá của các thuật toán best-first:

$$
\text{GBFS: } f(n) = h(n) \qquad
\text{A*: } f(n) = g(n) + h(n) \qquad
\text{Weighted A*: } f(n) = g(n) + w\,h(n)
$$

Hai heuristic chính, với $(x_n, y_n)$ là vị trí nút và $(x_g, y_g)$ là vị trí đích:

$$
h_M(n) = |x_n - x_g| + |y_n - y_g| \qquad
h_E(n) = \sqrt{(x_n - x_g)^2 + (y_n - y_g)^2}
$$

### 1.1. Tổ chức mã nguồn và các thay đổi trong notebook

Mỗi notebook **tự chứa** toàn bộ code của nó (không import module ngoài, trừ `maze_helper.py` có sẵn), để chạy lại được từ đầu bằng *Restart & Run All*. Thư viện dùng: `numpy`, `matplotlib`, `pandas` (chỉ để hiển thị bảng), `heapq` và `collections` (thư viện chuẩn). Không dùng thư viện cài đặt sẵn thuật toán tìm kiếm.

| Notebook | Bài | Những gì đã thêm hoặc sửa |
|---|---|---|
| `06_Explore_heuristics.ipynb` | 6 | Giữ nguyên các cell gốc. Thêm vào cuối: hàm `search()` (GBFS/A\* có tham số hòa điểm), `make_heuristics()`, `show_runs()`, thí nghiệm Manhattan/Euclidean, thí nghiệm hòa điểm, heuristic landmark (`bfs_distances()`, `landmark_heuristic()`), bảng so sánh trên 6 mê cung và phần trả lời. |
| `05_Maze_Example-Animation_Cycle_checking.ipynb` | 7 | Thay cell `import tree_search_solution as ts` (file không được phát hành) bằng cài đặt `DFS()` với 3 phương án cycle checking. Sửa 3 cell `ts.DFS(...)` thành `DFS(...)`. Thêm vào cuối: bảng so sánh trên open maze, biểu đồ frontier/bộ nhớ theo vòng lặp, bảng trên 8 mê cung và phần nhận xét. |
| `Ex1_Maze.ipynb` | 8 | Điền mọi cell `# Your code goes here` / `# Discussion`: định nghĩa bài toán, BFS, DFS, GBFS, A\*, thí nghiệm, bảng, biểu đồ, thảo luận, IDS, nhiều đích, Weighted A\*, các câu hỏi mở rộng. |
| `Ex2_Maze_simplified.ipynb` | 8 (rút gọn) | Điền Task 1, BFS, bảng kết quả, hình vẽ và 3 câu thảo luận. |

Thứ tự sinh hành động cố định là N, E, S, W, nên mọi kết quả (trừ thời gian chạy) tái lập được giữa các lần chạy.

---

## 2. Bài 6 – Ảnh hưởng của heuristic

### 2.1. Thí nghiệm đã chạy

Trên mê cung rỗng $10 \times 10$ (start $(1,1)$, goal $(8,8)$, $C^* = 14$) và trên các mê cung bài tập, mình chạy GBFS và A\* (graph search có `reached`, kiểm tra đích khi lấy nút ra khỏi frontier) với các heuristic:

- Manhattan,
- Euclidean,
- Landmark (ALT) – heuristic đề xuất, xem 2.5,
- $h^*$ (khoảng cách thật, tính bằng BFS ngược từ G) – dùng làm mốc lý tưởng.

Khi nhiều nút có cùng giá trị $f$, mặc định chọn nút được thêm **sau cùng** (LIFO). Riêng mục 2.2 so sánh thêm với FIFO.

### 2.2. Quan sát bản đồ $h(n)$ và $f(n)$ trong mê cung rỗng

**Bản đồ $h_M$ (GBFS).** Giá trị giảm đều đúng 1 đơn vị sau mỗi bước tiến về G, tạo thành một "dốc" rõ ràng. GBFS đi theo dốc này thẳng tới đích: chỉ mở rộng 14 nút, bằng đúng số bước của lời giải.

**Bản đồ $f = g + h_M$ (A\*).** Mọi ô trong hình chữ nhật có hai góc là S và G đều có:

$$
f(n) = g(n) + h_M(n) = 14 = C^*
$$

Vùng này là một **cao nguyên (plateau)**. Dù $h_M = h^*$ (heuristic hoàn hảo trên mê cung rỗng), $f$ vẫn không phân biệt được ô nào "tốt hơn". Hiệu quả khi đó phụ thuộc hoàn toàn vào cách xử lý hòa điểm:

| Cấu hình A\* (mê cung rỗng 10×10) | Chi phí | Số nút mở rộng |
|---|---|---|
| Manhattan, hòa điểm FIFO (nút cũ trước) | 14 | **63** (gần như cả hình chữ nhật) |
| Manhattan, hòa điểm LIFO (nút mới trước) | 14 | **14** |
| $(1 + 1/p)\cdot$Manhattan, FIFO, $p = 100$ | 14 | **14** |
| Euclidean, FIFO | 14 | 74 |
| Euclidean, LIFO | 14 | 61 |

Ưu tiên nút mới nhất, hoặc nhân $h$ với một hệ số rất nhỏ $1 + \varepsilon$, đều tạo thêm độ dốc nhỏ để A\* đi thẳng tới đích.

**Bản đồ $f = g + h_E$ (A\* + Euclidean).** $h_E$ đánh giá thấp chi phí thật, và thiếu hụt lớn nhất nằm dọc đường chéo. Kết quả là $f$ thấp nhất quanh S rồi **tăng dần** về phía G: dốc hướng sai chiều. A\* luôn phải mở rộng mọi nút có $f(n) < C^*$, mà vùng này gần như phủ toàn bộ khu vực giữa S và G (vùng đen trong hình cuối phần gốc của notebook).

### 2.3. So sánh Manhattan và Euclidean cho GBFS và A\*

| Mê cung | Heuristic | GBFS: chi phí | GBFS: số nút | A\*: chi phí | A\*: số nút |
|---|---|---|---|---|---|
| rỗng 10×10 | Manhattan | 14 | 14 | 14 | **14** |
| | Euclidean | 14 | 14 | 14 | 61 |
| empty_maze | Manhattan | 14 | 14 | 14 | **14** |
| | Euclidean | 14 | 14 | 14 | 61 |
| L_maze | Manhattan | 16 | 33 | 16 | **40** |
| | Euclidean | 20 | 33 | 16 | 62 |
| open_maze | Manhattan | 54 | 211 | 54 | **211** |
| | Euclidean | 54 | 54 | 54 | 516 |
| medium_maze | Manhattan | 152 | 155 | 68 | **217** |
| | Euclidean | 152 | 159 | 68 | 224 |
| loops_maze | Manhattan | 41 | 51 | 23 | **51** |
| | Euclidean | 35 | 51 | 23 | 57 |

**Nhận xét về GBFS.**

- Trên mê cung rỗng, cả hai heuristic dẫn GBFS thẳng tới đích (14 nút).
- Khi có tường, GBFS **không bảo đảm tối ưu** với cả hai heuristic: medium cho 152 bước (tối ưu 68); loops cho 41 bước với Manhattan, 35 với Euclidean (tối ưu 23); L maze với Euclidean cho 20 bước (tối ưu 16).
- Kết quả của GBFS phụ thuộc vào việc heuristic có "lừa" tác tử vào ngõ cụt hay không. Trên open maze, GBFS-Euclidean tình cờ đi đúng hướng ngay (54 nút), còn GBFS-Manhattan bị kéo vào căn phòng phía trên (211 nút). Nghĩa là heuristic tốt hơn về lý thuyết không bảo đảm GBFS tốt hơn trong mọi trường hợp.

**Nhận xét về A\*.**

- Cả hai heuristic đều chấp nhận được, nên A\* **luôn cho chi phí tối ưu**.
- A\* + Manhattan mở rộng ít nút hơn hoặc bằng ở mọi mê cung: 14 so với 61 (rỗng), 40 so với 62 (L), 211 so với 516 (open).
- Lý do là quan hệ trội (dominance):

$$
h_E(n) \;\le\; h_M(n) \;\le\; h^*(n)
$$

  A\* mở rộng mọi nút có $f(n) < C^*$, tức mọi nút có $h(n) < C^* - g(n)$. Heuristic lớn hơn (mà vẫn ≤ $h^*$) thì tập này nhỏ hơn. Do đó A\* với heuristic trội hơn không bao giờ mở rộng nhiều nút hơn, ngoại trừ chênh lệch nhỏ do cách xử lý hòa điểm.

### 2.4. Vì sao Manhattan phù hợp hơn Euclidean với lưới 4 hướng?

1. **Đúng với mô hình chuyển trạng thái.** Mỗi hành động N/E/S/W chỉ thay đổi hàng **hoặc** cột 1 đơn vị. Muốn đi từ $n$ tới $G$ phải thực hiện ít nhất $|\Delta r|$ bước dọc và $|\Delta c|$ bước ngang, nên khi không có tường:

$$
h^*(n) = |\Delta r| + |\Delta c| = h_M(n)
$$

   Manhattan là **chi phí chính xác của bài toán nới lỏng "bỏ tường"**. Đó là heuristic chặt nhất có thể nếu không xét đến tường.

2. **Euclidean là bài toán nới lỏng "quá mức".** Nó giả định tác tử đi theo đường thẳng, kể cả đi chéo, điều không thể làm trên lưới 4 hướng. Vì vậy Euclidean nhỏ hơn đáng kể. Theo đúng đường chéo ($|\Delta r| = |\Delta c| = k$):

$$
h_E = k\sqrt{2} = \frac{h_M}{\sqrt{2}} \approx 0.71\,h_M
$$

   tức đánh giá thấp gần 30% chi phí thật.

3. **Cả hai đều nhất quán**, nhưng Manhattan sát với $h^*$ hơn nên cung cấp nhiều thông tin hơn. Như đã giải thích ở 2.2, Euclidean còn tạo ra dốc $f$ ngược hướng.

4. **Khi nào Euclidean hợp lý?** Khi tác tử được di chuyển tự do theo mọi hướng (không gian liên tục, robot, bản đồ địa lý). Với lưới 8 hướng, heuristic phù hợp là Chebyshev hoặc octile distance.

### 2.5. Heuristic đề xuất: Landmark (ALT / differential heuristic)

**Ý tưởng.** Manhattan và Euclidean đều bỏ qua tường. Trong mê cung có ngõ cụt (L, open, medium), chúng kéo tác tử về phía đích theo đường chim bay và bắt A\* khảo sát các ngõ cụt. Heuristic landmark đưa thông tin về tường vào $h$:

1. Chọn trước $k$ ô mốc $L_1, \dots, L_k$. Trong cài đặt dùng $k = 2$, chọn theo kiểu "điểm xa nhất": mốc đầu tiên là ô xa G nhất, mốc tiếp theo là ô xa nhất so với các mốc đã chọn.
2. Với mỗi mốc, chạy BFS một lần để có khoảng cách thật $d(L, \cdot)$ tới mọi ô.
3. Theo bất đẳng thức tam giác, $d(L, n) \le d(L, G) + d(G, n)$ và $d(L, G) \le d(L, n) + d(n, G)$, nên:

$$
h_{ALT}(n) = \max_{L} \big|\, d(L, G) - d(L, n) \,\big| \;\le\; d(n, G) = h^*(n)
$$

4. Kết hợp với Manhattan:

$$
h(n) = \max\big( h_M(n),\; h_{ALT}(n) \big)
$$

**Tính chất.**

- *Chấp nhận được:* max của các heuristic chấp nhận được vẫn chấp nhận được.
- *Nhất quán:* $|d(L,G) - d(L,n)|$ thay đổi tối đa 1 khi đi một bước (vì $|d(L,n) - d(L,n')| \le 1$), và max của các heuristic nhất quán vẫn nhất quán.
- *Trội hơn Manhattan:* $h \ge h_M$ theo định nghĩa.

**Kết quả thực nghiệm (A\*).**

| Mê cung | Manhattan | Landmark (ALT) | $h^*$ (lý tưởng) | Chi phí |
|---|---|---|---|---|
| rỗng / empty_maze | 14 | 14 | 14 | 14 (tối ưu) |
| L_maze | 40 | **29** | 16 | 16 (tối ưu) |
| open_maze | 211 | **54** | 54 | 54 (tối ưu) |
| medium_maze | 217 | **82** | 68 | 68 (tối ưu) |
| loops_maze | 51 | **23** | 23 | 23 (tối ưu) |

ALT giảm số nút mở rộng từ 1.4 đến 4 lần, lời giải vẫn tối ưu. Trên mê cung rỗng nó trùng với Manhattan vì không có tường để khai thác.

**Cái giá.** Phải tiền xử lý $O(k \cdot n)$ thời gian và bộ nhớ ($k$ lần BFS, mỗi lần lưu một bảng khoảng cách). Cách này đáng giá khi cùng một bản đồ được dùng cho nhiều truy vấn (nhiều cặp S–G), như trong game hay bản đồ dẫn đường.

**Phương án phụ: Manhattan × $(1 + 1/p)$ để phá cao nguyên.** Chọn $p$ lớn hơn độ dài mọi đường đi có thể (ví dụ $p$ = số ô trống) và dùng $h(n) = (1 + 1/p)\,h_M(n)$. Về hình thức, heuristic này không chấp nhận được. Nhưng Weighted A\* bảo đảm:

$$
\text{cost} \;\le\; (1 + 1/p)\, C^* \;<\; C^* + 1 \quad (\text{vì } C^* < p)
$$

Chi phí là số nguyên nên $\text{cost} = C^*$: lời giải **vẫn tối ưu**, đồng thời cao nguyên $f = C^*$ bị phá (63 → 14 nút trên mê cung rỗng).

### 2.6. Điều kiện để heuristic dùng trong A\* vẫn bảo đảm tối ưu

1. **Không âm và bằng 0 tại đích:** $h(n) \ge 0$, $h(G) = 0$.

2. **Chấp nhận được (admissible):**

$$
h(n) \le h^*(n) \quad \forall n
$$

   Heuristic không bao giờ đánh giá cao chi phí thật. Điều kiện này đủ để A\* dạng **tree search** tối ưu (hoặc graph search có cho phép mở lại nút khi tìm được đường rẻ hơn).
   *Phác thảo chứng minh:* giả sử A\* trả về đích $G_2$ với $g(G_2) > C^*$. Khi đó trên đường tối ưu luôn có một nút $n$ còn trong frontier với $f(n) = g(n) + h(n) \le g(n) + h^*(n) = C^* < g(G_2) = f(G_2)$. Vậy $n$ phải được chọn trước $G_2$, mâu thuẫn.

3. **Nhất quán (consistent / monotone):**

$$
h(n) \le c(n, a, n') + h(n') \quad \text{với mọi cạnh } (n, a, n')
$$

   - Nhất quán ⇒ chấp nhận được.
   - Nhất quán ⇒ $f$ không giảm dọc mọi đường đi. Do đó lần đầu một trạng thái được lấy ra khỏi frontier là đã có đường tối ưu tới nó, và A\* **graph search với `reached` không cần mở lại nút** vẫn tối ưu.
   - Manhattan, Euclidean, ALT và max của chúng đều nhất quán trên lưới 4 hướng với chi phí bước 1.

4. **Lưu ý về chất lượng và chi phí:**
   - Trong các heuristic chấp nhận được, heuristic **trội hơn** (gần $h^*$ hơn) thì A\* mở rộng ít nút hơn.
   - Thời gian tính $h$ phải rẻ so với số nút tiết kiệm được. $h^*$ tính bằng BFS là ví dụ cực đoan: A\* chỉ mở rộng đúng đường đi tối ưu, nhưng tính $h^*$ đã tốn bằng một lần tìm kiếm đầy đủ.
   - Nếu dùng $w\,h$ với $w > 1$ (Weighted A\*) thì mất bảo đảm tối ưu; chỉ còn $\text{cost} \le w\,C^*$.

### 2.7. Cài đặt (code trong `06_Explore_heuristics.ipynb`)

#### a) Thuật toán tìm kiếm dùng chung cho GBFS và A\*

Mê cung $10 \times 10$ của notebook không có tường bao quanh, nên `neighbors()` kiểm tra cả biên lưới lẫn tường. Hàm `search()` là best-first search dạng graph search:

- `g` lưu chi phí tốt nhất đã biết tới mỗi ô, `parent` lưu ô cha để truy hồi đường đi.
- Frontier là heap gồm các bộ `(f, khóa hòa điểm, ô)`. Khóa hòa điểm là `-count` (LIFO: nút mới ưu tiên) hoặc `count` (FIFO: nút cũ ưu tiên). Đây là cách duy nhất thay đổi giữa hai chế độ.
- GBFS không mở lại ô đã gặp. A\* cập nhật ô nếu tìm được $g$ nhỏ hơn (`closed.discard(t)` cho phép mở rộng lại).
- Kiểm tra đích khi lấy ô ra khỏi heap, như yêu cầu để A\* tối ưu.

```python
def neighbors(grid, pos):
    rows_, cols_ = grid.shape
    for dr, dc in MOVES:
        r, c = pos[0] + dr, pos[1] + dc
        if 0 <= r < rows_ and 0 <= c < cols_ and grid[r, c] != "X":
            yield (r, c)
```

```python
def search(grid, start, goal, h, strategy="A*", tie="LIFO"):
    """GBFS (f = h) hoặc A* (f = g + h). Trả về chi phí, số nút mở rộng và các ô đã mở rộng."""
    start, goal = tuple(int(v) for v in start), tuple(int(v) for v in goal)
    count = 0
    g = {start: 0}
    parent = {start: None}
    frontier = [(h(start), 0, start)]
    expanded = []
    closed = set()
    while frontier:
        _, _, s = heapq.heappop(frontier)
        if s in closed:
            continue
        if s == goal:
            path = [s]
            while parent[path[-1]] is not None:
                path.append(parent[path[-1]])
            return {"cost": g[s], "expanded": len(expanded), "explored": expanded, "path": path[::-1]}
        closed.add(s)
        expanded.append(s)
        for t in neighbors(grid, s):
            new_g = g[s] + 1
            if t not in g or (strategy == "A*" and new_g < g[t]):
                g[t], parent[t] = new_g, s
                closed.discard(t)
                count += 1
                f = h(t) if strategy == "GBFS" else new_g + h(t)
                heapq.heappush(frontier, (f, -count if tie == "LIFO" else count, t))
    return {"cost": None, "expanded": len(expanded), "explored": expanded, "path": []}
```

Hai heuristic được tạo dưới dạng closure theo vị trí đích:

```python
def make_heuristics(goal):
    gr, gc = int(goal[0]), int(goal[1])
    return {
        "Manhattan": lambda p: abs(p[0] - gr) + abs(p[1] - gc),
        "Euclidean": lambda p: ((p[0] - gr) ** 2 + (p[1] - gc) ** 2) ** 0.5,
    }
```

`show_runs()` vẽ nhiều lần chạy cạnh nhau: xám là ô đã mở rộng, đỏ là đường đi, chữ S/G đánh dấu điểm đầu và đích.

#### b) Thí nghiệm Manhattan/Euclidean và thí nghiệm hòa điểm

Thí nghiệm chính chạy mọi tổ hợp {GBFS, A\*} × {Manhattan, Euclidean} trên mê cung rỗng:

```python
H = make_heuristics(goal)
runs = {f"{alg} – {name}": search(maze, start, goal, h, alg)
        for alg in ["GBFS", "A*"] for name, h in H.items()}
```

Thí nghiệm hòa điểm so sánh cùng heuristic Manhattan với các cách phá hòa khác nhau, cộng thêm $(1 + 1/p)\cdot h_M$ với $p = n^2 = 100$:

```python
manh = H["Manhattan"]
eps = 1 / (n * n)     # 1/p với p = số ô, lớn hơn mọi độ dài đường đi
tie_runs = {
    "A* Manhattan, FIFO": search(maze, start, goal, manh, "A*", tie="FIFO"),
    "A* Manhattan, LIFO": search(maze, start, goal, manh, "A*", tie="LIFO"),
    "A* (1+1/p)·Manhattan, FIFO": search(maze, start, goal, lambda p: (1 + eps) * manh(p), "A*", tie="FIFO"),
    "A* Euclidean, FIFO": search(maze, start, goal, H["Euclidean"], "A*", tie="FIFO"),
}
show_runs(maze, start, goal, tie_runs)
```

#### c) Heuristic landmark (ALT)

`bfs_distances()` tính khoảng cách thật từ một ô tới mọi ô (BFS trên lưới). `landmark_heuristic()` chọn $k$ mốc theo kiểu "điểm xa nhất" rồi trả về closure tính $\max(h_M, \max_L |d(L,G) - d(L,n)|)$:

```python
def bfs_distances(grid, source):
    """Khoảng cách thật từ source tới mọi ô (BFS trên lưới)."""
    source = tuple(int(v) for v in source)
    dist = {source: 0}
    queue = deque([source])
    while queue:
        s = queue.popleft()
        for t in neighbors(grid, s):
            if t not in dist:
                dist[t] = dist[s] + 1
                queue.append(t)
    return dist
```

```python
def landmark_heuristic(grid, goal, k=2):
    """h(n) = max(Manhattan, max_L |d(L,G) - d(L,n)|) với k mốc chọn theo kiểu "farthest point"."""
    goal = tuple(int(v) for v in goal)
    manh = make_heuristics(goal)["Manhattan"]
    landmarks, tables = [], []
    far = bfs_distances(grid, goal)
    for _ in range(k):
        # mốc tiếp theo: ô xa nhất so với các mốc đã chọn (và đích)
        lm = max(far, key=far.get)
        d = bfs_distances(grid, lm)
        landmarks.append(lm)
        tables.append(d)
        far = {s: min(far[s], d[s]) for s in far}
    return lambda p: max(manh(p), *(abs(d[goal] - d.get(p, 0)) for d in tables)), landmarks
```

Thí nghiệm cuối chạy GBFS và A\* với 4 heuristic (Manhattan, Euclidean, ALT, $h^*$) trên 6 mê cung. $h^*$ lấy chính là bảng `bfs_distances` tính từ đích:

```python
from collections import deque


def bfs_distances(grid, source):
    """Khoảng cách thật từ source tới mọi ô (BFS trên lưới)."""
    source = tuple(int(v) for v in source)
    dist = {source: 0}
    queue = deque([source])
    while queue:
        s = queue.popleft()
        for t in neighbors(grid, s):
            if t not in dist:
                dist[t] = dist[s] + 1
                queue.append(t)
    return dist


def landmark_heuristic(grid, goal, k=2):
    """h(n) = max(Manhattan, max_L |d(L,G) - d(L,n)|) với k mốc chọn theo kiểu "farthest point"."""
    goal = tuple(int(v) for v in goal)
    manh = make_heuristics(goal)["Manhattan"]
    landmarks, tables = [], []
    far = bfs_distances(grid, goal)
    for _ in range(k):
        # mốc tiếp theo: ô xa nhất so với các mốc đã chọn (và đích)
        lm = max(far, key=far.get)
        d = bfs_distances(grid, lm)
        landmarks.append(lm)
        tables.append(d)
        far = {s: min(far[s], d[s]) for s in far}
    return lambda p: max(manh(p), *(abs(d[goal] - d.get(p, 0)) for d in tables)), landmarks


def load(file):
    with open(file) as f:
        grid = np.array([list(row) for row in f.read().split("\n") if row])
    s = np.argwhere(grid == "S")[0]
    g = np.argwhere(grid == "G")[0]
    return grid, tuple(int(v) for v in s), tuple(int(v) for v in g)


test_mazes = {"empty 10x10 (above)": (maze, tuple(int(v) for v in start), tuple(int(v) for v in goal))}
for file in ["empty_maze.txt", "L_maze.txt", "open_maze.txt", "medium_maze.txt", "loops_maze.txt"]:
    test_mazes[file.replace(".txt", "")] = load(file)

records = []
alt_runs = {}
for name, (grid, s, gp) in test_mazes.items():
    hs = make_heuristics(gp)
    hs["Landmark (ALT)"], _ = landmark_heuristic(grid, gp)
    exact = bfs_distances(grid, gp)
    hs["h* (exact)"] = lambda p, exact=exact: exact[p]
    for hname, h in hs.items():
        for alg in ["GBFS", "A*"]:
            r = search(grid, s, gp, h, alg)
            records.append({"maze": name, "heuristic": hname, "algorithm": alg,
                         "cost": r["cost"], "expanded": r["expanded"]})
            if name == "L_maze" and alg == "A*":
                alt_runs[f"A* – {hname}"] = r

res_df = pd.DataFrame(records)
res_df.pivot_table(index=["maze", "heuristic"], columns="algorithm", aggfunc="first",
                   values=["cost", "expanded"], sort=False)
```

---

## 3. Bài 7 – Kiểm tra chu trình trong DFS

### 3.1. Bối cảnh và cài đặt

DFS cài đặt đúng cách **không dùng `reached`**, để giữ bộ nhớ $O(bm)$. Khi đó nó chỉ biết các nút đang nằm trong cây tìm kiếm hiện tại, nên phải tự xử lý chu trình. Nếu không, DFS sẽ đi vòng vô hạn trong mê cung có vòng lặp hoặc vùng trống.

Notebook gốc gọi `tree_search_solution.py`, file không được phát hành, nên mình cài đặt lại DFS với ba phương án. Tham số `frontier_option` giữ như notebook gốc. Kiểm chứng: PA3 tạo ra **đúng chuỗi 54 hành động** giống output gốc (21×W, 17×S, 13×W, 3×S).

| `frontier_option` | Phương án | Khi sinh con có trạng thái $s$ |
|---|---|---|
| 3 | **PA1** – kiểm tra đường đi | Bỏ $s$ nếu $s$ nằm trên đường đi từ gốc tới nút hiện tại; ngược lại luôn đẩy vào stack (có thể trùng) |
| 2 | **PA2** – đường đi + không thêm trùng frontier | Như PA1, và bỏ $s$ nếu $s$ đang có trong frontier |
| 1 | **PA3** – đường đi + đưa lên đỉnh stack | Như PA1; nếu $s$ đang có trong frontier thì xóa bản ghi cũ và đặt bản ghi mới lên đỉnh stack |

Chi tiết cài đặt chung:

- Bảng `in_frontier` chỉ chứa các trạng thái **đang** ở trong frontier, nên kích thước vẫn là $O(bm)$; đây không phải `reached`.
- Mỗi nút đếm số con còn sống. Khi một nhánh không còn con nào trong frontier, cả nhánh được giải phóng khỏi bộ nhớ.
- Với PA3, việc "xóa bản ghi cũ" làm theo kiểu *xóa lười*: chỉ đánh dấu `removed`, rồi bỏ qua khi bản ghi đó được lấy ra.

### 3.2. Kết quả trên `open_maze`

| Chỉ số | PA1 (đường đi) | PA2 (+ không trùng) | PA3 (+ lên đỉnh) |
|---|---|---|---|
| Giải được? | Không | Không | **Có** |
| Chi phí đường đi | N/A | N/A | 54 |
| Số lần lặp (lấy khỏi stack) | 200 000 (chạm `max_tries`) | 200 000 (chạm `max_tries`) | **344** |
| Số lần mở rộng | 200 000 | 200 000 | 211 |
| Số ô khác nhau đã mở rộng | 179 | 125 | 211 |
| **Số lần mở rộng lại** (lặp trạng thái) | **199 821** | **199 875** | **0** |
| Kích thước frontier lớn nhất | 155 | 93 | **64** |
| Số bản ghi trùng lớn nhất trong frontier | 24 | 0 | 0 |
| Số nút tối đa trong bộ nhớ | 333 | 201 | 333 |

Với `max_tries = 500` (mức dùng cho animation), PA1 và PA2 cũng không giải được: lần lượt có 321 và 383 lần mở rộng lại chỉ trong 500 vòng lặp.

Kết quả của cả ba phương án trên tất cả mê cung (`max_tries = 200 000`):

| Mê cung | PA1: chi phí | PA2: chi phí | PA3: chi phí | PA1: mở rộng lại | PA2: mở rộng lại | PA3: mở rộng lại |
|---|---|---|---|---|---|---|
| small | 29 | 29 | 29 | 0 | 0 | 0 |
| medium | 152 | 152 | 152 | 0 | 0 | 0 |
| large | 210 | 210 | 210 | 0 | 0 | 0 |
| loops | 27 | 27 | 27 | 0 | 0 | 0 |
| L | 122 | 80 | 122 | 0 | 5 | 0 |
| empty | 80 | 54 | 80 | 0 | 7 | 0 |
| empty_2 | 34 | **N/A** | 34 | 0 | 199 933 | 0 |
| open | **N/A** | **N/A** | 54 | 199 821 | 199 875 | 0 |

### 3.3. Phân tích từng phương án

**PA1 – chỉ kiểm tra đường đi hiện tại.**

- *Ưu điểm:* chặn được chu trình trên một nhánh. Mọi nhánh là một đường đơn, độ sâu ≤ $n - 1$, nên về lý thuyết DFS không lặp vô hạn và đầy đủ trong không gian hữu hạn.
- *Vấn đề:* PA1 không biết gì về các nhánh khác.
  - Cùng một ô có thể được đẩy vào stack nhiều lần từ các ô lân cận khác nhau (tối đa 24 bản ghi trùng).
  - Sau khi một nhánh được giải phóng, các ô của nó lại được thăm theo một **đường đơn khác**.
  - Trong một căn phòng trống, số đường đơn tăng theo hàm mũ với diện tích phòng. Vì vậy DFS "tô kín" căn phòng (animation: phòng đỏ đặc), rồi quét đi quét lại: gần 200 000 lần mở rộng lại mà không ra khỏi phòng.
- Về lý thuyết PA1 vẫn sẽ kết thúc, nhưng sau một số bước theo hàm mũ, nên trên thực tế coi như lặp vô hạn.

**PA2 – thêm điều kiện không đưa trạng thái đã có trong frontier vào lại.**

- *Ưu điểm:* frontier gọn hơn (93 so với 155), không có bản ghi trùng, và tốn ít bộ nhớ nhất (201 nút).
- *Vấn đề:* bản ghi được giữ lại là bản ghi **cũ**, thuộc một nhánh **nông hơn**.
  - DFS đi sâu theo nhánh hiện tại; các ô kề đã có bản ghi cũ nên không được thêm lại.
  - Khi nhánh hiện tại xong và bị giải phóng (ô trắng lại), các bản ghi cũ vẫn nằm dưới stack (ô cam còn sót trong animation).
  - Khi lấy ra, chúng khiến DFS **khám phá lại toàn bộ vùng vừa xóa**. Hiện tượng này lặp lại liên tục.
- Kết quả: thất bại trên open maze và cả empty_maze_2. Ở L và empty maze, PA2 lại tình cờ ra đường ngắn hơn (80 và 54 bước), nhưng đã phải mở rộng lại vài ô. Hành vi của PA2 phụ thuộc may rủi vào thứ tự các bản ghi cũ còn sót.

**PA3 – đưa trạng thái tìm thấy lên đỉnh stack.**

- Mỗi trạng thái chỉ có **một** bản ghi trong frontier, và bản ghi đó luôn thuộc **nhánh đang được đi sâu**: nhánh mới nhất "nhận" luôn các ô kề của nó.
- Khi một nhánh được giải phóng, frontier không còn bản ghi nào của vùng đó (không còn ô cam). DFS "biết" vùng này đã xong và không quay lại.
- Kết quả: trên open maze, mỗi ô chỉ được mở rộng đúng một lần (211 lần mở rộng, 0 lần mở rộng lại), giải được sau 344 vòng lặp, frontier tối đa chỉ 64.
- Bộ nhớ tối đa (333 nút) bằng PA1, vì vẫn phải giữ một đường đi rất sâu (độ sâu tới 178).

**Các mê cung khác.**

- Mê cung dạng hành lang (small, medium, large, loops): gần như không có hai đường khác nhau tới cùng một ô, nên cả ba phương án giống nhau.
- L và empty maze: theo thứ tự hướng NESW, DFS chạm đích trước khi phải quay lui qua vùng trống, nên PA1 và PA3 trùng kết quả.
- Vấn đề chỉ lộ rõ ở **open maze**, nơi đích nằm sau một căn phòng lớn mà DFS phải quay lui ra.

### 3.4. So sánh tổng hợp

| Tiêu chí | PA1 | PA2 | PA3 |
|---|---|---|---|
| Số lần lặp (open) | vượt `max_tries` | vượt `max_tries` | 344 (giải được) |
| Kích thước frontier | lớn nhất, có bản ghi trùng | nhỏ hơn, không trùng | không trùng, nhỏ nhất |
| Lặp trạng thái | rất nhiều | rất nhiều | không |
| Chi phí quản lý | kiểm tra đường đi $O(m)$ | + bảng băm `in_frontier` | + bảng băm + xóa lười trong stack |
| Dùng được trong IDS? | Có, đầy đủ và tối ưu | Không ổn định | Có thể bỏ sót lời giải (xem 4.6) |

### 3.5. Đánh đổi của cycle checking

Cycle checking làm giảm lặp trạng thái nhưng tăng chi phí quản lý dữ liệu:

- **Kiểm tra đường đi** tốn $O(m)$ cho mỗi con được sinh ra (phải đi ngược lên các nút cha). Với đường đi sâu ~200 bước, đây là chi phí đáng kể.
- **PA2 và PA3** cần thêm cấu trúc tra cứu "trạng thái nào đang ở frontier", tức một bảng băm cập nhật mỗi lần push/pop.
- **PA3** cần xóa một phần tử ở giữa stack. Xóa thật tốn $O(|frontier|)$. Xóa lười thì mỗi thao tác là $O(1)$, nhưng stack chứa các bản ghi chết, tốn bộ nhớ và thêm vòng lặp để bỏ qua chúng.
- **Giới hạn cơ bản:** không phương án nào loại được hoàn toàn đường đi dư thừa như `reached` của BFS. Muốn làm được thì phải lưu mọi trạng thái đã gặp, và khi đó DFS mất lợi thế bộ nhớ $O(bm)$, trở thành $O(n)$ như BFS.

### 3.6. Cài đặt (code trong `05_Maze_Example-Animation_Cycle_checking.ipynb`)

#### a) Lớp `Node` và kiểm tra chu trình

`Node` có thêm hai trường so với lớp gốc:

- `n_children`: số con còn sống trong cây. Khi về 0 thì nút có thể được giải phóng.
- `removed`: đánh dấu bản ghi frontier đã bị thay thế (xóa lười, dùng cho PA3).

`set_order()` giữ giao diện của notebook gốc: thứ tự đẩy con vào stack, con cuối cùng được lấy ra trước. `on_path()` đi ngược lên các nút cha để kiểm tra chu trình, tốn $O(\text{độ sâu})$.

```python
def set_order(new_order="NESW"):
    """Thứ tự đẩy các con vào stack (con cuối cùng được lấy ra trước)."""
    global order
    order = list(new_order)
    print("Directions are checked in the order", order)
```

```python
class Node:
    def __init__(self, pos, parent, action, cost):
        self.pos = tuple(pos)
        self.parent = parent
        self.action = action
        self.cost = cost
        self.n_children = 0      # số con còn sống trong cây
        self.removed = False     # True: bản ghi frontier đã bị thay thế (PA3)

    def get_path_from_root(self):
        node, path = self, [self]
        while node.parent is not None:
            node = node.parent
            path.append(node)
        return path[::-1]
```

```python
def on_path(node, pos):
    while node is not None:
        if node.pos == pos:
            return True
        node = node.parent
    return False
```

#### b) Hàm `DFS()` với 3 phương án cycle checking

Các cấu trúc dữ liệu chính:

| Biến | Vai trò |
|---|---|
| `stack` | Frontier LIFO; có thể chứa bản ghi đã bị đánh dấu `removed` |
| `in_frontier` | `trạng thái → bản ghi hợp lệ` đang trong frontier; dùng cho PA2/PA3; kích thước $O(bm)$ |
| `counts` | Số bản ghi hợp lệ của mỗi trạng thái trong frontier; dùng để đo số bản ghi trùng |
| `alive` | Số nút đang được giữ trong bộ nhớ (cây tìm kiếm hiện tại) |
| `explored` | **Chỉ để thống kê** số lần mở rộng lại; thuật toán không đọc tập này |

Luồng xử lý mỗi vòng lặp:

1. Lấy bản ghi ở đỉnh stack. Nếu nó đã bị `removed` thì giải phóng và bỏ qua.
2. Nếu là đích thì trả kết quả.
3. Mở rộng: với mỗi hướng theo `order`, bỏ con nếu là tường hoặc nằm trên đường đi (`on_path`). Sau đó:
   - `frontier_option = 3` (PA1): luôn đẩy vào stack;
   - `frontier_option = 2` (PA2): bỏ nếu trạng thái đã có trong `in_frontier`;
   - `frontier_option = 1` (PA3): đánh dấu bản ghi cũ `removed`, rồi đẩy bản ghi mới lên đỉnh.
4. Nếu nút không sinh được con nào, gọi `release()` để giải phóng nó và các tổ tiên không còn con.
5. Cập nhật thống kê: kích thước frontier, số bản ghi trùng (`tổng bản ghi − số trạng thái khác nhau`), bộ nhớ, và lưu vết theo từng vòng lặp để vẽ biểu đồ.

Khi `animation=True`, mỗi `frame_every` vòng lặp hàm `frame()` vẽ một khung hình. Ô đỏ là các nút còn trong bộ nhớ, tìm bằng cách đi ngược từ mọi bản ghi frontier lên gốc. Ô cam là frontier. Khung cuối cùng chỉ vẽ đường đi tìm được. Kết quả trả về theo định dạng mà `animate_maze()` của `maze_helper.py` cần (`path`, `reached`, `actions`, `maze_anim`), cộng thêm các thống kê.

```python
def DFS(maze, max_tries=500, frontier_option=1, animation=False, frame_every=5, vis=False):
    """DFS không reached với 3 phương án cycle checking (xem bảng ở trên)."""
    start = find_pos(maze, "S")
    start = (int(start[0]), int(start[1]))
    root = Node(start, None, None, 0)
    stack = [root]
    in_frontier = {start: root}
    counts = Counter({start: 1})        # số bản ghi hợp lệ của mỗi trạng thái trong frontier
    alive = 1
    expanded = reexpanded = 0
    explored = set()                    # chỉ để thống kê, thuật toán không đọc
    stats = {"max_frontier": 1, "max_duplicates": 0, "max_memory": 1}
    trace = {"frontier": [], "memory": []}
    frames = []

    def release(node):
        nonlocal alive
        while node is not None and node.n_children == 0:
            alive -= 1
            node = node.parent
            if node is not None:
                node.n_children -= 1

    def frame(current=None):
        m = np.copy(maze)
        live = [n for n in stack if not n.removed] + ([current] if current else [])
        seen = set()
        for n in live:
            while n is not None and id(n) not in seen:
                seen.add(id(n))
                if m[n.pos] == " ":
                    m[n.pos] = "P"
                n = n.parent
        for n in stack:
            if not n.removed and m[n.pos] in " P":
                m[n.pos] = "F"
        frames.append(m)

    def finish(goal_node, tries):
        res = {"path": None, "actions": None, "reached": [], "maze_anim": frames,
               "option": OPTION_NAMES[frontier_option], "tries": tries,
               "expanded": expanded, "distinct_expanded": len(explored),
               "reexpanded": reexpanded, "trace": trace, **stats}
        if goal_node is not None:
            nodes = goal_node.get_path_from_root()
            res["path"] = [n.pos for n in nodes]
            res["actions"] = [n.action for n in nodes[1:]]
            if animation:
                m = np.copy(maze)
                for p in res["path"]:
                    if m[p] == " ":
                        m[p] = "P"
                frames.append(m)
        return res

    tries = 0
    while stack:
        tries += 1
        if tries > max_tries:
            return finish(None, tries - 1)
        node = stack.pop()
        if node.removed:
            release(node)
            continue
        counts[node.pos] -= 1
        if in_frontier.get(node.pos) is node:
            del in_frontier[node.pos]

        if animation and tries % frame_every == 1:
            frame(node)
        if maze[node.pos] == "G":
            return finish(node, tries)

        expanded += 1
        reexpanded += node.pos in explored
        explored.add(node.pos)
        for a in order:
            dr, dc = DIRECTIONS[a]
            s = (node.pos[0] + dr, node.pos[1] + dc)
            if maze[s] == "X" or on_path(node, s):
                continue
            if s in in_frontier and frontier_option != 3:
                if frontier_option == 2:
                    continue
                old = in_frontier[s]          # frontier_option == 1
                old.removed = True
                counts[s] -= 1
            child = Node(s, node, a, node.cost + 1)
            node.n_children += 1
            alive += 1
            stack.append(child)
            in_frontier[s] = child
            counts[s] += 1
        if node.n_children == 0:
            release(node)

        live = sum(counts.values())
        distinct = sum(1 for v in counts.values() if v > 0)
        stats["max_frontier"] = max(stats["max_frontier"], live)
        stats["max_duplicates"] = max(stats["max_duplicates"], live - distinct)
        stats["max_memory"] = max(stats["max_memory"], alive)
        trace["frontier"].append(live)
        trace["memory"].append(alive)

    return finish(None, tries)
```

#### c) Chạy animation và thí nghiệm so sánh

Ba cell animation của notebook gốc được giữ nguyên ý nghĩa, chỉ đổi lời gọi. `frame_every = 5` giữ mỗi animation khoảng 100 khung hình để notebook không quá nặng:

```python
%time result = DFS(maze, max_tries=500, frontier_option=3, animation=True)   # PA1
animate_maze(result)
```

Thí nghiệm so sánh chạy cả 3 phương án với `max_tries` = 500 và 200 000, rồi tổng hợp bằng `summarize()`:

```python
import pandas as pd


def summarize(res, max_tries):
    return {
        "option": res["option"], "max_tries": max_tries,
        "solved": res["path"] is not None,
        "path cost": len(res["path"]) - 1 if res["path"] else "N/A",
        "tries": res["tries"], "expanded": res["expanded"],
        "distinct": res["distinct_expanded"], "re-expanded": res["reexpanded"],
        "max frontier": res["max_frontier"], "max duplicates": res["max_duplicates"],
        "max memory": res["max_memory"],
    }


open_runs = {}
rows = []
for max_tries in [500, 200_000]:
    for opt in [3, 2, 1]:
        res = DFS(maze, max_tries=max_tries, frontier_option=opt)
        open_runs[(opt, max_tries)] = res
        rows.append(summarize(res, max_tries))
pd.DataFrame(rows).set_index(["max_tries", "option"])
```

Biểu đồ vẽ `res["trace"]["frontier"]` và `res["trace"]["memory"]` trong 2 000 vòng lặp đầu của mỗi phương án. Bảng trên 8 mê cung lặp cùng quy trình với `max_tries = 200 000`:

```python
maze_files = ["small_maze.txt", "medium_maze.txt", "large_maze.txt", "open_maze.txt",
              "L_maze.txt", "loops_maze.txt", "empty_maze.txt", "empty_maze_2.txt"]
rows = []
for file in maze_files:
    with open(file, "r") as f:
        m = parse_maze(f.read())
    for opt in [3, 2, 1]:
        res = DFS(m, max_tries=200_000, frontier_option=opt)
        rows.append({"maze": file.replace(".txt", ""), "option": res["option"],
                     "path cost": len(res["path"]) - 1 if res["path"] else "N/A",
                     "tries": res["tries"], "re-expanded": res["reexpanded"],
                     "max frontier": res["max_frontier"], "max memory": res["max_memory"]})
all_df = pd.DataFrame(rows)
all_df.pivot(index="maze", columns="option", values=["path cost", "tries", "re-expanded"])
```

---

## 4. Bài 8 – Goal-based Planning Agent

### 4.1. Task 1 – Mô hình hóa bài toán tìm kiếm

Môi trường **quan sát đầy đủ, rời rạc, tất định và đã biết**. Tác tử có bản đồ, nên có thể lập toàn bộ kế hoạch trước (offline), rồi thực hiện theo kế hoạch mà không cần đọc percept (open-loop).

| Thành phần | Định nghĩa |
|---|---|
| **Trạng thái** | $s = (r, c)$: vị trí của tác tử trên một ô không phải tường |
| **Trạng thái đầu** | $s_0$ = vị trí ô `S` |
| **Hành động** | $A = \{N, E, S, W\}$; $\text{ACTIONS}(s)$ = các hướng mà ô kế tiếp không phải `X` |
| **Mô hình chuyển trạng thái** | $\text{RESULT}((r,c),N) = (r-1,c)$; $S \to (r+1,c)$; $E \to (r,c+1)$; $W \to (r,c-1)$ |
| **Kiểm tra đích** | $\text{IS-GOAL}(s) \iff \text{maze}[s] = \texttt{G}$ (kiểm tra theo ký tự nên hỗ trợ nhiều đích) |
| **Chi phí đường đi** | $c(s, a, s') = 1$; chi phí đường đi = số bước = độ sâu của nút |

Các hàm cài đặt chỉ **đọc** bản đồ. Mọi thông tin tìm kiếm (nút, frontier, reached) nằm trong cây tìm kiếm, đúng yêu cầu "không lưu thông tin vào bản đồ".

**Ước lượng kích thước bài toán:**

| Đại lượng | Cách xác định |
|---|---|
| $n$ | Đếm số ô không phải tường |
| $d$ | Không biết trước; tính chính xác bằng BFS (tối ưu vì chi phí bước bằng nhau) |
| $m$ | $\infty$ nếu không kiểm tra chu trình; nếu có thì mọi nhánh là đường đơn, nên $m \le n - 1$ |
| $b$ | $\max_s |\text{ACTIONS}(s)|$; tối đa 4. Trong hành lang, hệ số hiệu dụng chỉ khoảng 1–2 |

| | small | medium | large | open | L | loops | empty | empty_2 |
|---|---|---|---|---|---|---|---|---|
| $n$ | 94 | 274 | 647 | 684 | 157 | 72 | 100 | 100 |
| $d$ | 19 | 68 | 210 | 54 | 16 | 23 | 14 | 14 |
| $m$ (cận trên) | 93 | 273 | 646 | 683 | 156 | 71 | 99 | 99 |
| $b$ | 3 | 3 | 4 | 4 | 4 | 4 | 4 | 4 |

### 4.2. Cài đặt các thuật toán (tóm tắt)

| Thuật toán | Frontier | Phát hiện lặp | Kiểm tra đích |
|---|---|---|---|
| BFS | FIFO queue (`deque`) | `reached` (set) | Sớm, khi sinh nút con (AIMA) |
| DFS | LIFO stack | Không `reached`; kiểm tra chu trình theo đường đi + PA3; giải phóng nhánh đã xong | Khi lấy nút ra |
| GBFS | Priority queue theo $h$ | `reached`, không mở lại nút | Khi lấy nút ra |
| A\* | Priority queue theo $g + h$ | `reached`, cập nhật nếu tìm được $g$ nhỏ hơn | Khi lấy nút ra (cần cho tối ưu) |

Heuristic là Manhattan tới đích gần nhất. Khi hòa điểm, ưu tiên nút được thêm sau cùng.

### 4.3. BFS và DFS xử lý vòng lặp (chu trình) như thế nào?

**BFS** dùng `reached`, tập các trạng thái đã sinh ra. Một trạng thái chỉ vào frontier khi được sinh lần đầu. Mọi đường khác tới trạng thái đó, kể cả chu trình quay lại, đều bị loại. Như vậy BFS loại được cả **chu trình** lẫn **đường đi dư thừa** và không bao giờ lặp trạng thái. Cái giá là phải lưu toàn bộ `reached`.

**DFS không có `reached`** chỉ biết cây tìm kiếm hiện có trong bộ nhớ, nên chỉ có thể **kiểm tra chu trình**: không sinh con có trạng thái đã nằm trên đường đi từ gốc tới nút đang mở rộng (chi phí $O(\text{độ sâu})$). Cách này đủ để không lặp vô hạn, nhưng không chặn được đường dư thừa: một ô vẫn có thể được thăm lại từ nhánh khác. Vì vậy cài đặt dùng thêm PA3 (Bài 7) để DFS không quét lại vùng trống.

### 4.4. Tính đầy đủ, tối ưu và độ phức tạp

| | BFS | DFS (không `reached`) | GBFS | A\* (Manhattan) |
|---|---|---|---|---|
| **Đầy đủ** | Có: hữu hạn + `reached` | Có trong không gian hữu hạn nhờ kiểm tra chu trình; thực tế cần PA3 hoặc `max_tries` | Có: hữu hạn + `reached` | Có: chi phí bước > 0, hữu hạn |
| **Tối ưu** | Có: chi phí bước bằng nhau, duyệt theo lớp | Không: trả lời giải đầu tiên trên nhánh sâu | Không: bỏ qua $g(n)$ | Có: $h$ chấp nhận được và nhất quán |
| **Thời gian** | $O(b^d)$; với `reached` ≤ $O(n)$ | $O(b^m)$, $m \le n-1$; mỗi lần kiểm tra chu trình tốn thêm $O(m)$ | $O(b^m)$; với `reached` ≤ $O(n \log n)$ | $O(b^d)$; với `reached` ≤ $O(n \log n)$ |
| **Bộ nhớ** | $O(b^d)$, ≤ $O(n)$ | **$O(bm)$** | $O(n)$ | $O(n)$ |

**Vì sao các thuật toán đầy đủ / tối ưu (hoặc không):**

- **BFS tối ưu** vì mọi nút ở độ sâu $k$ đều được mở rộng trước mọi nút ở độ sâu $k+1$. Mọi bước có cùng chi phí 1, nên lời giải nông nhất cũng là lời giải rẻ nhất. Kiểm tra đích sớm vẫn đúng vì mọi nút cùng độ sâu có cùng chi phí.
- **DFS không tối ưu:** DFS đi sâu theo thứ tự hành động cố định và dừng ở đích đầu tiên gặp. Ví dụ L maze: 122 bước, tối ưu là 16.
- **GBFS không tối ưu:** GBFS chỉ nhìn $h$, không quan tâm đã đi bao xa. Ví dụ medium: 152 bước, tối ưu 68; loops: 41 bước, tối ưu 23.
- **A\* tối ưu:** Manhattan chấp nhận được ($h \le h^*$, vì mỗi bước giảm khoảng cách Manhattan tối đa 1) và nhất quán ($h(n) \le 1 + h(n')$). Lần đầu một trạng thái được lấy ra khỏi frontier là đã có đường tối ưu tới nó.

**Khác biệt về bộ nhớ giữa BFS và DFS:**

- BFS phải nhớ mọi trạng thái đã gặp, nên bộ nhớ tỉ lệ với vùng đã khảo sát.
- DFS chỉ giữ đường đi hiện tại cùng các nút anh em chưa thăm của mỗi nút trên đường đi ($b$ nút × $m$ tầng), và giải phóng nhánh đã xong.
- Thực nghiệm:
  - large maze: BFS giữ tối đa 622 nút, DFS chỉ 251, dù đường đi gần như dài bằng nhau (210 bước);
  - medium: 270 so với 160.
- Lợi thế này **biến mất khi $m$ lớn**: empty maze DFS đi 80 bước và giữ 153 nút, BFS chỉ 96; L maze 231 so với 152. Bộ nhớ $O(bm)$ chỉ nhỏ khi đường đi DFS không quá dài.

### 4.5. Task 4 – Kết quả thực nghiệm

(Thời gian tính bằng ms, chỉ để tham khảo, thay đổi theo máy.)

**Small maze** ($d = 19$)

| Thuật toán | Chi phí | Nút mở rộng | Độ sâu max | Nút trong bộ nhớ max | Frontier max |
|---|---|---|---|---|---|
| BFS | 19 | 90 | 19 | 93 | 9 |
| DFS | 29 | 39 | 29 | 34 | 5 |
| GBFS | 29 | 39 | 29 | 44 | 5 |
| A\* | 19 | 51 | 19 | 58 | 7 |

**Medium maze** ($d = 68$)

| Thuật toán | Chi phí | Nút mở rộng | Độ sâu max | Nút trong bộ nhớ max | Frontier max |
|---|---|---|---|---|---|
| BFS | 68 | 267 | 68 | 270 | 9 |
| DFS | 152 | 153 | 152 | 160 | 8 |
| GBFS | 152 | 155 | 152 | 163 | 8 |
| A\* | 68 | 217 | 68 | 224 | 8 |

**Large maze** ($d = 210$, chỉ có một lời giải)

| Thuật toán | Chi phí | Nút mở rộng | Độ sâu max | Nút trong bộ nhớ max | Frontier max |
|---|---|---|---|---|---|
| BFS | 210 | 618 | 210 | 622 | 8 |
| DFS | 210 | 337 | 214 | 251 | 38 |
| GBFS | 210 | 456 | 210 | 481 | 25 |
| A\* | 210 | 542 | 210 | 553 | 12 |

**Open maze** ($d = 54$)

| Thuật toán | Chi phí | Nút mở rộng | Độ sâu max | Nút trong bộ nhớ max | Frontier max |
|---|---|---|---|---|---|
| BFS | 54 | 679 | 54 | 683 | 25 |
| DFS (PA3) | 54 | 211 | 178 | 333 | 64 |
| GBFS | 54 | 211 | 67 | 275 | 64 |
| A\* | 54 | 211 | 54 | 275 | 185 |

**L maze** ($d = 16$)

| Thuật toán | Chi phí | Nút mở rộng | Độ sâu max | Nút trong bộ nhớ max | Frontier max |
|---|---|---|---|---|---|
| BFS | 16 | 146 | 16 | 152 | 17 |
| DFS | 122 | 132 | 122 | 231 | 23 |
| GBFS | 16 | 33 | 16 | 64 | 31 |
| A\* | 16 | 40 | 16 | 69 | 29 |

**Loops maze** ($d = 23$)

| Thuật toán | Chi phí | Nút mở rộng | Độ sâu max | Nút trong bộ nhớ max | Frontier max |
|---|---|---|---|---|---|
| BFS | 23 | 70 | 23 | 72 | 8 |
| DFS | 27 | 30 | 27 | 39 | 11 |
| GBFS | 41 | 51 | 41 | 61 | 10 |
| A\* | 23 | 51 | 23 | 61 | 11 |

**Empty maze** ($d = 14$)

| Thuật toán | Chi phí | Nút mở rộng | Độ sâu max | Nút trong bộ nhớ max | Frontier max |
|---|---|---|---|---|---|
| BFS | 14 | 92 | 14 | 96 | 12 |
| DFS | 80 | 80 | 80 | 153 | 12 |
| GBFS | 14 | 14 | 14 | 43 | 29 |
| A\* | 14 | 14 | 14 | 43 | 29 |

**Empty maze 2 (xoay)** ($d = 14$)

| Thuật toán | Chi phí | Nút mở rộng | Độ sâu max | Nút trong bộ nhớ max | Frontier max |
|---|---|---|---|---|---|
| BFS | 14 | 92 | 14 | 96 | 11 |
| DFS | 34 | 34 | 34 | 76 | 32 |
| GBFS | 14 | 14 | 14 | 43 | 29 |
| A\* | 14 | 14 | 14 | 43 | 29 |

### 4.6. Những bài học quan trọng

1. **Tối ưu:** BFS và A\* luôn cho đường ngắn nhất; DFS và GBFS thì không. Chênh lệch lớn nhất: L maze (DFS 122 / 16), empty (DFS 80 / 14), medium (DFS và GBFS 152 / 68), loops (GBFS 41 / 23).
2. **Heuristic thu hẹp vùng khảo sát:** A\* mở rộng ít nút hơn BFS ở mọi mê cung (open 211 / 679; L 40 / 146; empty 14 / 92). Ở mê cung chỉ có một lời giải (large), mọi thuật toán phải đi gần hết hành lang, nên lợi ích nhỏ (542 / 618).
3. **Bộ nhớ:** DFS tiết kiệm bộ nhớ chỉ khi đường đi của nó không quá dài (large: 251 / 622 của BFS). Khi DFS đi vòng vèo (empty, L), $m$ lớn và lợi thế mất đi.
4. **Frontier của A\* có thể lớn:** open maze A\* có frontier tối đa 185, lớn nhất trong các thuật toán. Hàng đợi ưu tiên giữ lại nhiều nút "đang chờ" trong căn phòng có $f$ gần bằng nhau.
5. **Cycle checking trong vùng trống rất khó:** chỉ kiểm tra đường đi là không đủ trên thực tế. Phải quản lý frontier (PA3) thì DFS mới giải được open maze.
6. **Cách xử lý hòa điểm và thứ tự sinh con rất quan trọng:** ưu tiên nút mới nhất giúp A\* và GBFS đi thẳng trong mê cung rỗng (14 nút thay vì 63). Thứ tự hướng đi quyết định gần như toàn bộ hành vi của DFS.

### 4.7. Phần nâng cao – IDS

IDS gọi DFS giới hạn độ sâu (DLS) với $\ell = 0, 1, 2, \dots$ cho tới khi tìm thấy lời giải. Mỗi lần lặp được phép tối đa 1 000 000 lần lấy nút.

| Mê cung | Kiểm tra chu trình | Chi phí | $d$ tối ưu | Giới hạn cuối | Tổng nút mở rộng | Bộ nhớ max |
|---|---|---|---|---|---|---|
| small | đường đi (PA1) | 19 | 19 | 19 | 923 | 25 |
| | PA3 | 29 | 19 | 29 | 1 583 | 34 |
| medium | PA1 | 68 | 68 | 68 | 15 665 | 75 |
| | PA3 | 70 | 68 | 70 | 9 594 | 77 |
| large | PA1 | 210 | 210 | 210 | 60 179 | 248 |
| | PA3 | 210 | 210 | 210 | 60 179 | 248 |
| open | PA1 | **N/A\*** | 54 | 15 | 650 515 | 43 |
| | PA3 | 54 | 54 | 54 | 1 009 028 | 137 |
| L | PA1 | **N/A\*** | 16 | 14 | 827 299 | 37 |
| | PA3 | 42 | 16 | 43 | 16 555 | 97 |
| loops | PA1 | 23 | 23 | 23 | 60 701 | 36 |
| | PA3 | 27 | 23 | 27 | 1 279 | 39 |
| empty | PA1 | 14 | 14 | 14 | 297 368 | 33 |
| | PA3 | 20 | 14 | 21 | 1 401 | 48 |
| empty_2 | PA1 | 14 | 14 | 14 | 204 382 | 39 |
| | PA3 | 16 | 14 | 16 | 1 565 | 42 |

**Nhận xét:**

- **IDS + kiểm tra chu trình theo đường đi (PA1)** là một DLS đầy đủ ở mỗi lần lặp, nên IDS **đầy đủ và tối ưu**: mọi mê cung giải được đều có chi phí đúng bằng $d$. Bộ nhớ chỉ $O(bd)$ (medium: 75 nút so với 270 của BFS).
- **Cái giá là thời gian.** Mỗi lần tăng giới hạn phải tìm lại từ đầu. Quan trọng hơn, khi không có `reached`, số đường đơn có độ dài ≤ $\ell$ tăng theo hàm mũ trong vùng trống.
  - Empty maze cần ~300 000 lần mở rộng, BFS chỉ cần 92.
  - Open maze ($d = 54$) và L maze: chỉ một lần lặp ở độ sâu 14–15 đã vượt 1 000 000 bước, nên ghi **N/A\***.
  - Đây là giới hạn của thuật toán (không `reached` thì không loại được đường dư thừa), không phải lỗi cài đặt.
- **IDS + PA3** chạy nhanh và giải được mọi mê cung, nhưng **mất tính tối ưu** (small 29, L 42, empty 20, loops 27).
  - Lý do: PA3 thay bản ghi **nông** của một trạng thái bằng bản ghi **sâu hơn**. Khi có giới hạn độ sâu, bản ghi sâu có thể bị cắt tại $\ell$, nên DLS bỏ sót lời giải ở đúng độ sâu đó.
  - Ví dụ L maze: có lời giải dài 42, nhưng DLS(42) không tìm thấy; phải tới DLS(43) mới tìm thấy.
  - Đây chính là lưu ý trong đề: *"IDS phụ thuộc vào cơ chế kiểm tra chu trình của DFS"*.

### 4.8. Phần nâng cao – Bài toán nhiều đích

Thêm đích vào medium maze. Tác tử hoàn thành khi tới **bất kỳ** ô G nào, nên lời giải tối ưu là đường tới **đích gần nhất**.

| Mê cung | Thuật toán | Chi phí | Đích đạt được | Nút mở rộng | Bộ nhớ max |
|---|---|---|---|---|---|
| medium + 1 đích (9,33) | BFS | **23** | (9,33) | 57 | 60 |
| | DFS | 152 | (16,1) | 153 | 160 |
| | IDS | **23** | (9,33) | 624 | 26 |
| | A\* | **23** | (9,33) | 30 | 33 |
| medium + 2 đích (5,20), (13,1) | BFS | **24** | (5,20) | 61 | 64 |
| | DFS | 107 | (13,1) | 108 | 112 |
| | IDS | **24** | (5,20) | 662 | 27 |
| | A\* | **24** | (5,20) | 42 | 45 |

**Giải thích:**

- **BFS và IDS tối ưu:** cả hai duyệt theo độ sâu tăng dần, nên đích đầu tiên gặp là đích nông nhất. Chúng không cần biết có bao nhiêu đích hay đích ở đâu.
- **DFS không tối ưu:** DFS dừng ở đích đầu tiên trên nhánh đang đi sâu. Ở ví dụ 1, nó bỏ qua đích (9,33) cách start 23 bước và đi 152 bước tới đích gốc.
- **A\* vẫn tối ưu** khi dùng $h(n) = \min_{g \in G} h_M(n, g)$. Min của các heuristic chấp nhận được (mỗi cái ≤ chi phí tới đích tương ứng) vẫn ≤ chi phí tới đích gần nhất, và vẫn nhất quán. Nếu chỉ dùng khoảng cách tới **một** đích cố định, $h$ có thể vượt chi phí thật tới đích gần hơn, và A\* mất tính tối ưu.

### 4.9. Phần nâng cao (không bắt buộc)

**Giao lộ làm trạng thái.** Khi chỉ dùng các giao lộ làm trạng thái, mỗi cạnh nối hai giao lộ có chi phí bằng độ dài hành lang giữa chúng, nên chi phí bước **không còn bằng nhau**. BFS và IDS đếm theo số cạnh nên không còn tối ưu. Cần thay bằng uniform-cost search (frontier ưu tiên theo $g(n)$), hoặc iterative lengthening (tăng dần giới hạn theo chi phí thay vì độ sâu).

**Weighted A\*.** Đã chạy với $f = g + w\,h$:

| $w$ | medium: chi phí / nút | open: chi phí / nút | L: chi phí / nút | empty: chi phí / nút |
|---|---|---|---|---|
| 0.5 | 68 / 241 | 54 / 603 | 16 / 124 | 14 / 90 |
| 1 (A\*) | 68 / 217 | 54 / 211 | 16 / 40 | 14 / 14 |
| 1.5 | 68 / 229 | 54 / 329 | 16 / 37 | 14 / 14 |
| 2 | 68 / 227 | 54 / 435 | 16 / 40 | 14 / 14 |
| 5 | **102** / 226 | 54 / 633 | 16 / 42 | 14 / 14 |
| 100 | **152** / 157 | 54 / 717 | 16 / 44 | 14 / 14 |

- $w < 1$: gần uniform-cost search (BFS). Vẫn tối ưu nhưng mở rộng nhiều nút hơn (open 603, empty 90).
- $w > 1$: heuristic bị "thổi phồng", không còn chấp nhận được hay nhất quán; chỉ bảo đảm $\text{cost} \le w\,C^*$. Ở medium, chất lượng giảm rõ: 68 → 102 → 152, tiến gần GBFS.
- Số nút mở rộng **không phải lúc nào cũng giảm**. Ở open maze, heuristic kéo tác tử vào căn phòng ngõ cụt; vì $h$ không còn nhất quán, A\* phải mở lại các trạng thái khi tìm được đường rẻ hơn (435 nút với $w = 2$, 717 với $w = 100$). Lợi ích tốc độ chỉ rõ khi heuristic chỉ đúng hướng (L maze: 40 → 37 nút với $w = 1.5$, chi phí vẫn 16).

**Mê cung chưa biết (unknown maze).**

- Môi trường trở thành *quan sát một phần* và *chưa biết*: tác tử không có mô hình chuyển trạng thái, nên không thể lập kế hoạch trước mà phải **xen kẽ lập kế hoạch và hành động** (online search).
- **PEAS:**
  - *Performance:* số bước tới G, có thể kèm số lần va tường.
  - *Environment:* mê cung chưa biết, tĩnh, tất định, rời rạc.
  - *Actuators:* di chuyển N/E/S/W.
  - *Sensors:* vị trí hiện tại, tường ở 4 ô kề, đã ở G hay chưa.
- **Tác tử hợp lý:** tác tử dựa trên mô hình, ghi nhớ phần bản đồ đã khám phá và chạy online DFS (khám phá, quay lui bằng hành động ngược lại) hoặc LRTA\* (cập nhật $h$ của các ô đã đi qua). Một cách khác là lập kế hoạch trên bản đồ "lạc quan" (coi ô chưa biết là trống) và lập lại kế hoạch mỗi khi phát hiện tường.
- **Có GPS (biết khoảng cách tới đích):** dùng khoảng cách đó làm heuristic cho LRTA\* hoặc A\* lạc quan, để ưu tiên khám phá về phía đích và giảm đáng kể quãng đường khám phá.

### 4.10. Cài đặt (code trong `Ex1_Maze.ipynb`)

Code dùng lớp `Node` có sẵn trong notebook (`pos`, `parent`, `action`, `cost`, `get_path_from_root()`). Riêng DFS gắn thêm thuộc tính `n_children` và `removed` lên từng nút khi cần.

#### a) Task 1 – Các thành phần của bài toán

Mỗi thành phần của bài toán được cài thành một hàm, đặt theo tên trong AIMA. `find_all()` trả về **mọi** vị trí của một ký tự, nên cùng một code chạy được cho bài toán nhiều đích.

```python
def find_all(maze, what):
    """Tất cả vị trí (row, col) chứa ký tự `what` (dùng cho bài toán nhiều đích)."""
    return [tuple(int(v) for v in p) for p in zip(*np.where(maze == what))]
```

```python
def actions(maze, pos):
    """ACTIONS(s): các hướng đi không dẫn vào tường."""
    r, c = pos
    return [a for a, (dr, dc) in DIRECTIONS.items() if maze[r + dr, c + dc] != "X"]
```

```python
def result(pos, action):
    """RESULT(s, a): mô hình chuyển trạng thái."""
    dr, dc = DIRECTIONS[action]
    return (pos[0] + dr, pos[1] + dc)
```

```python
def is_goal(maze, pos):
    """IS-GOAL(s)."""
    return maze[pos] == "G"
```

```python
def expand(maze, node):
    """EXPAND(problem, node): sinh các nút con, mỗi bước có chi phí 1."""
    for a in actions(maze, node.pos):
        yield Node(result(node.pos, a), node, a, node.cost + 1)
```

Tải 8 mê cung và đếm $n$, $b$, $m$. Sau khi có BFS, notebook thêm cột $d$ bằng `bfs(m)["path_cost"]`:

```python
import pandas as pd

MAZE_FILES = {
    "small": "small_maze.txt", "medium": "medium_maze.txt", "large": "large_maze.txt",
    "open": "open_maze.txt", "L": "L_maze.txt", "loops": "loops_maze.txt",
    "empty": "empty_maze.txt", "empty_2": "empty_maze_2.txt",
}


def load_maze(file):
    with open(file, "r") as f:
        return mh.parse_maze(f.read())


MAZES = {name: load_maze(file) for name, file in MAZE_FILES.items()}


def problem_size(maze):
    free = [p for p in zip(*np.where(maze != "X"))]
    return {"n": len(free),
            "b": max(len(actions(maze, p)) for p in free),
            "m (cận trên)": len(free) - 1}


pd.DataFrame({name: problem_size(m) for name, m in MAZES.items()}).T
```

#### b) Task 2 – BFS và DFS

`make_result()` đóng gói kết quả của mọi thuật toán theo cùng một định dạng: đường đi, chuỗi hành động, chi phí, các thống kê cần cho bảng (số nút mở rộng, độ sâu max, frontier max, số nút trong bộ nhớ max, thời gian), cùng tập ô đã mở rộng và frontier cuối (chỉ để vẽ).

```python
def make_result(goal_node, expanded, max_depth, max_frontier, max_memory,
                explored, frontier_states, start_time, **extra):
    """Đóng gói kết quả và các thống kê của một lần tìm kiếm."""
    res = {
        "path": None, "actions": None, "path_cost": None,
        "expanded": expanded,            # số nút được mở rộng
        "max_depth": max_depth,          # độ sâu lớn nhất của cây tìm kiếm
        "max_frontier": max_frontier,    # kích thước frontier lớn nhất
        "max_memory": max_memory,        # số nút tối đa cùng lúc trong bộ nhớ
        "explored": explored,            # chỉ để vẽ hình
        "frontier": frontier_states,     # chỉ để vẽ hình
        "time_ms": (time.perf_counter() - start_time) * 1000,
    }
    if goal_node is not None:
        nodes = goal_node.get_path_from_root()
        res["path"] = [n.pos for n in nodes]
        res["actions"] = [n.action for n in nodes[1:]]
        res["path_cost"] = goal_node.cost
    res.update(extra)
    return res
```

**BFS** theo đúng mã giả AIMA (Fig. 3.9):

- frontier là `deque` (FIFO), `reached` là `set`;
- kiểm tra đích **sớm**, ngay khi sinh nút con;
- "nút trong bộ nhớ" chính là kích thước `reached`.

```python
def bfs(maze):
    """Breadth-first search theo mã giả AIMA (Fig. 3.9).

    frontier: FIFO queue; reached: tập trạng thái đã sinh ra; kiểm tra đích sớm
    (early goal test) khi sinh nút con.
    """
    t0 = time.perf_counter()
    start = find_all(maze, "S")[0]
    node = Node(start, None, None, 0)
    if is_goal(maze, start):
        return make_result(node, 0, 0, 1, 1, set(), [], t0)

    frontier = deque([node])
    reached = {start}
    expanded = max_depth = 0
    max_frontier = 1
    explored = set()

    while frontier:
        node = frontier.popleft()
        expanded += 1
        explored.add(node.pos)
        for child in expand(maze, node):
            s = child.pos
            if is_goal(maze, s):
                max_depth = max(max_depth, child.cost)
                return make_result(child, expanded, max_depth, max_frontier, len(reached) + 1,
                                   explored, [n.pos for n in frontier], t0)
            if s not in reached:
                reached.add(s)
                frontier.append(child)
                max_depth = max(max_depth, child.cost)
        max_frontier = max(max_frontier, len(frontier))

    return make_result(None, expanded, max_depth, max_frontier, len(reached), explored, [], t0)
```

**DFS** không dùng `reached`:

- Kiểm tra chu trình bằng `on_path()`.
- Quản lý frontier theo PA3 (mặc định `frontier_option="move_top"`) để không quét lại vùng trống.
- Giải phóng nhánh đã xong bằng `release()`. `alive` đếm số nút thực sự còn trong cây, nên "max # of nodes in memory" phản ánh đúng bộ nhớ $O(bm)$.
- Tham số `limit` biến DFS thành Depth-Limited Search để dùng lại cho IDS. Khi một nút chạm giới hạn, nó bị cắt (`cutoff = True`) và giải phóng.
- `max_tries` chặn số vòng lặp. Nếu vượt, trả `status = "max_tries"` (ghi N/A\* trong bảng).
- Kết quả có `status` thuộc một trong các giá trị `solved`, `cutoff`, `failure`, `max_tries` để IDS biết khi nào dừng.

```python
def on_path(node, pos):
    """Kiểm tra chu trình: pos có nằm trên đường đi từ gốc tới node không? O(độ sâu)."""
    while node is not None:
        if node.pos == pos:
            return True
        node = node.parent
    return False
```

```python
def dfs(maze, limit=None, max_tries=100_000, frontier_option="move_top"):
    """Depth-first search KHÔNG dùng reached, có kiểm tra chu trình và giải phóng bộ nhớ.

    frontier_option (chiến lược kiểm tra chu trình, xem 05_..._Cycle_checking.ipynb):
      "path"     : chỉ bỏ các con đã xuất hiện trên đường đi hiện tại.
      "no_dup"   : như "path" + không thêm trạng thái đang có trong frontier.
      "move_top" : như "path" + nếu trạng thái đã có trong frontier thì đưa nó lên đỉnh stack.
    limit: giới hạn độ sâu (Depth-Limited Search). None = không giới hạn.
    max_tries: số lần lấy nút tối đa để tránh vòng lặp quá dài.
    """
    t0 = time.perf_counter()
    start = find_all(maze, "S")[0]
    root = Node(start, None, None, 0)
    root.n_children = 0              # số con còn sống -> dùng để giải phóng nhánh đã xong

    stack = [root]                   # frontier LIFO
    in_frontier = {start: root}      # trạng thái -> bản ghi hợp lệ trong frontier
    alive = 1                        # số nút đang được giữ trong bộ nhớ
    expanded = max_depth = 0
    max_frontier = max_memory = 1
    tries = 0
    cutoff = False
    explored = set()                 # CHỈ để vẽ hình, thuật toán không đọc tập này

    def release(node):
        """Giải phóng node và các tổ tiên không còn con nào trong cây."""
        nonlocal alive
        while node is not None and node.n_children == 0:
            alive -= 1
            node = node.parent
            if node is not None:
                node.n_children -= 1

    def live_frontier():
        return [n.pos for n in stack if not getattr(n, "removed", False)]

    while stack:
        tries += 1
        if tries > max_tries:
            return make_result(None, expanded, max_depth, max_frontier, max_memory, explored,
                               live_frontier(), t0, status="max_tries", tries=tries - 1)
        node = stack.pop()
        if getattr(node, "removed", False):
            release(node)            # bản ghi cũ đã được đưa lên đỉnh ở chỗ khác
            continue
        if in_frontier.get(node.pos) is node:
            del in_frontier[node.pos]

        if is_goal(maze, node.pos):
            return make_result(node, expanded, max_depth, max_frontier, max_memory, explored,
                               live_frontier(), t0, status="solved", tries=tries)
        if limit is not None and node.cost >= limit:
            cutoff = True
            release(node)
            continue

        expanded += 1
        explored.add(node.pos)
        for child in expand(maze, node):
            s = child.pos
            if on_path(node, s):
                continue                               # chu trình -> bỏ
            if s in in_frontier:
                if frontier_option == "no_dup":
                    continue
                if frontier_option == "move_top":
                    in_frontier[s].removed = True      # xóa lười bản ghi cũ
            child.n_children = 0
            node.n_children += 1
            alive += 1
            stack.append(child)
            in_frontier[s] = child
            max_depth = max(max_depth, child.cost)

        if node.n_children == 0:
            release(node)                              # lá cụt -> giải phóng ngay
        max_frontier = max(max_frontier, len(in_frontier) if frontier_option != "path" else len(stack))
        max_memory = max(max_memory, alive)

    return make_result(None, expanded, max_depth, max_frontier, max_memory, explored, [], t0,
                       status="cutoff" if cutoff else "failure", tries=tries)
```

**Hiển thị lời giải.** `solution_maze()` tạo **bản sao** mê cung và đánh dấu `.` (đã mở rộng), `F` (frontier), `P` (đường đi). Bản đồ gốc không bị sửa. `show_solutions()` vẽ các thuật toán cạnh nhau với cùng bảng màu như `mh.show_maze()`.

```python
def solution_maze(maze, res):
    """Bản sao của mê cung, đánh dấu ô đã mở rộng, frontier và đường đi."""
    m = np.copy(maze)
    for p in res["explored"]:
        if m[p] == " ":
            m[p] = "."
    for p in res["frontier"]:
        if m[p] in " .":
            m[p] = "F"
    for p in res["path"] or []:
        if m[p] not in "SG":
            m[p] = "P"
    return m
```

#### c) Task 3 – GBFS và A\*

`best_first_search()` theo AIMA Fig. 3.7, dùng chung cho GBFS, A\* và Weighted A\*; chỉ khác nhau ở hàm `f`:

- frontier là heap gồm `(f, -counter, node)`. `-counter` làm hòa điểm theo LIFO: nút thêm sau được ưu tiên, giúp giữ hướng đi.
- `reached` là dict `trạng thái → nút tốt nhất`. Một bản ghi trong heap bị coi là lỗi thời nếu nó không còn là nút trong `reached`, và bị bỏ qua khi lấy ra (thay cho thao tác decrease-key).
- GBFS không mở lại trạng thái đã gặp. A\* cập nhật khi tìm được $g$ nhỏ hơn.
- Kiểm tra đích khi lấy nút ra (bắt buộc để A\* tối ưu).
- Heuristic lấy **min theo mọi đích**, nên chạy đúng cho bài toán nhiều đích.

```python
def manhattan(pos, goals):
    """Khoảng cách Manhattan tới đích gần nhất."""
    return min(abs(pos[0] - g[0]) + abs(pos[1] - g[1]) for g in goals)
```

```python
def best_first_search(maze, strategy="A*", heuristic=manhattan, w=1.0):
    """Best-first search (AIMA Fig. 3.7) với hàm đánh giá:

      GBFS:        f(n) = h(n)
      A*:          f(n) = g(n) + h(n)
      Weighted A*: f(n) = g(n) + w * h(n)

    frontier: priority queue (heapq); reached: dict trạng thái -> nút tốt nhất.
    Kiểm tra đích khi lấy nút ra khỏi frontier (cần cho tính tối ưu của A*).
    Hòa điểm f: ưu tiên nút được thêm sau cùng (LIFO) để giữ hướng đi.
    """
    t0 = time.perf_counter()
    start = find_all(maze, "S")[0]
    goals = find_all(maze, "G")

    def f(node):
        h = heuristic(node.pos, goals)
        return h if strategy == "GBFS" else node.cost + w * h

    counter = 0
    node = Node(start, None, None, 0)
    frontier = [(f(node), -counter, node)]
    reached = {start: node}
    expanded = max_depth = 0
    max_frontier = 1
    explored = set()

    while frontier:
        _, _, node = heapq.heappop(frontier)
        if node is not reached[node.pos]:
            continue                       # bản ghi lỗi thời: đã có đường tốt hơn
        if is_goal(maze, node.pos):
            return make_result(node, expanded, max_depth, max_frontier, len(reached),
                               explored, [n.pos for _, _, n in frontier], t0)
        expanded += 1
        explored.add(node.pos)
        for child in expand(maze, node):
            s = child.pos
            # GBFS không quan tâm g(n) nên không mở lại trạng thái đã gặp;
            # A* cập nhật nếu tìm được đường rẻ hơn.
            better = strategy != "GBFS" and s in reached and child.cost < reached[s].cost
            if s not in reached or better:
                reached[s] = child
                counter += 1
                heapq.heappush(frontier, (f(child), -counter, child))
                max_depth = max(max_depth, child.cost)
        max_frontier = max(max_frontier, len(frontier))

    return make_result(None, expanded, max_depth, max_frontier, len(reached), explored, [], t0)
```

```python
def gbfs(maze):
    return best_first_search(maze, "GBFS")
```

```python
def astar(maze):
    return best_first_search(maze, "A*")
```

#### d) Task 4 – Thí nghiệm, bảng và biểu đồ

Chạy 4 thuật toán trên 8 mê cung, gom vào một `DataFrame`:

```python
ALGORITHMS = {
    "BFS": bfs,
    "DFS": dfs,
    "GBFS": gbfs,
    "A*": astar,
}

results = {m: {a: alg(MAZES[m]) for a, alg in ALGORITHMS.items()} for m in MAZES}


def to_row(maze_name, alg, res):
    return {
        "maze": maze_name, "algorithm": alg,
        "path cost": res["path_cost"] if res["path"] else "N/A*",
        "# of nodes expanded": res["expanded"],
        "max tree depth": res["max_depth"],
        "max # of nodes in memory": res["max_memory"],
        "max frontier size": res["max_frontier"],
        "time (ms)": round(res["time_ms"], 2),
    }


table = pd.DataFrame([to_row(m, a, r) for m, rs in results.items() for a, r in rs.items()])
table.set_index(["maze", "algorithm"])
```

Từ `table`, notebook:

- vẽ lời giải của 4 thuật toán cho từng mê cung (`show_solutions`);
- in **bảng Markdown cho từng mê cung**, sinh tự động từ `table` để số liệu luôn khớp với lần chạy (không cần thư viện `tabulate`);
- vẽ 4 biểu đồ cột nhóm (chi phí, số nút mở rộng, bộ nhớ, frontier) theo mê cung và thuật toán.

```python
metrics = ["path cost", "# of nodes expanded", "max # of nodes in memory", "max frontier size"]
fig, axes = plt.subplots(2, 2, figsize=(14, 8))
for ax, metric in zip(axes.flat, metrics):
    plot_df.pivot(index="maze", columns="algorithm", values=metric)[list(ALGORITHMS)] \
        .loc[list(MAZES)].plot.bar(ax=ax, rot=0)
```

#### e) Nâng cao – IDS

`ids()` gọi lại `dfs(limit=ℓ)` với $\ell = 0, 1, 2, \dots$ và cộng dồn số nút mở rộng. Vòng lặp dừng khi DLS trả về trạng thái khác `cutoff`: tìm thấy, thất bại hẳn, hoặc vượt `max_tries`. Notebook chạy IDS với cả hai cách kiểm tra chu trình (`path` = PA1, `move_top` = PA3) để thấy sự đánh đổi ở mục 4.7.

```python
def ids(maze, max_limit=1000, max_tries=1_000_000, frontier_option="path"):
    """Iterative deepening: gọi DFS giới hạn độ sâu với limit = 0, 1, 2, ...

    max_tries áp dụng cho từng lần lặp; nếu vượt thì dừng và báo "max_tries".
    """
    t0 = time.perf_counter()
    total_expanded = max_frontier = max_memory = 0
    for limit in range(max_limit + 1):
        res = dfs(maze, limit=limit, max_tries=max_tries, frontier_option=frontier_option)
        total_expanded += res["expanded"]
        max_frontier = max(max_frontier, res["max_frontier"])
        max_memory = max(max_memory, res["max_memory"])
        if res["status"] != "cutoff":          # tìm thấy, thất bại hẳn, hoặc vượt max_tries
            break
    res.update(expanded=total_expanded, max_frontier=max_frontier, max_memory=max_memory,
               time_ms=(time.perf_counter() - t0) * 1000, limit=limit)
    return res


ids_rows = []
for name, maze in MAZES.items():
    for option in ["path", "move_top"]:
        r = ids(maze, frontier_option=option)
        ids_rows.append({
            "maze": name, "cycle check": option,
            "path cost": r["path_cost"] if r["path"] else "N/A*",
            "optimal d": results[name]["BFS"]["path_cost"],
            "final limit": r["limit"], "status": r["status"],
            "# of nodes expanded": r["expanded"],
            "max # of nodes in memory": r["max_memory"],
            "max frontier size": r["max_frontier"],
            "time (ms)": round(r["time_ms"], 1),
        })
pd.DataFrame(ids_rows).set_index(["maze", "cycle check"])
```

#### f) Nâng cao – Nhiều đích

`add_goals()` tạo **bản sao** medium maze và đặt thêm ô `G` (có kiểm tra ô phải trống). Vì `is_goal()` kiểm tra theo ký tự và heuristic lấy min theo mọi đích, BFS, DFS, IDS và A\* chạy được mà không cần sửa code.

```python
def add_goals(maze, goals):
    """Bản sao của mê cung với thêm các ô đích."""
    m = np.copy(maze)
    for g in goals:
        assert m[g] == " ", f"{g} is not free"
        m[g] = "G"
    return m


medium = MAZES["medium"]
MULTI = {
    "medium + 1 goal (9,33)": add_goals(medium, [(9, 33)]),
    "medium + 2 goals (5,20), (13,1)": add_goals(medium, [(5, 20), (13, 1)]),
}

MULTI_ALGS = {"BFS": bfs, "DFS": dfs, "IDS": ids, "A*": astar}
multi_results = {name: {a: alg(m) for a, alg in MULTI_ALGS.items()} for name, m in MULTI.items()}

pd.DataFrame([
    {"maze": name, "algorithm": a, "path cost": r["path_cost"],
     "goal reached": r["path"][-1], "# of nodes expanded": r["expanded"],
     "max # of nodes in memory": r["max_memory"]}
    for name, rs in multi_results.items() for a, r in rs.items()
]).set_index(["maze", "algorithm"])
```

#### g) Nâng cao – Weighted A\*

Chỉ cần truyền tham số `w` cho `best_first_search()`:

```python
weights = [0.5, 1, 1.5, 2, 5, 100]
rows = []
for name in ["medium", "open", "L", "loops", "empty"]:
    for w in weights:
        r = best_first_search(MAZES[name], "A*", w=w)
        rows.append({"maze": name, "w": w, "path cost": r["path_cost"],
                     "# of nodes expanded": r["expanded"], "max frontier size": r["max_frontier"]})
wa = pd.DataFrame(rows)
wa.pivot(index="w", columns="maze", values=["path cost", "# of nodes expanded"])
```

### 4.11. Cài đặt `Ex2_Maze_simplified.ipynb` (chỉ BFS)

Bản rút gọn dùng lại `Node` của notebook và `mh.find_pos()`. BFS cùng thuật toán như Ex1 nhưng gọn hơn: trả về danh sách ô đã mở rộng và frontier để vẽ. Hình được vẽ bằng chính `mh.show_maze()` trên một bản sao đã đánh dấu.

```python
def actions(maze, pos):
    """Các hướng đi không dẫn vào tường."""
    r, c = pos
    return [a for a, (dr, dc) in DIRECTIONS.items() if maze[r + dr, c + dc] != "X"]
```

```python
def result(pos, action):
    dr, dc = DIRECTIONS[action]
    return (pos[0] + dr, pos[1] + dc)
```

```python
def bfs(maze):
    """Breadth-first search (AIMA): frontier FIFO, reached, kiểm tra đích sớm.

    Trả về dict gồm path, actions, path_cost, số nút mở rộng, tập ô đã mở rộng
    và frontier cuối cùng (hai tập này chỉ dùng để vẽ hình).
    """
    start = tuple(int(v) for v in mh.find_pos(maze, "S"))
    root = Node(start, None, None, 0)
    frontier = deque([root])
    reached = {start}
    expanded = []

    while frontier:
        node = frontier.popleft()
        expanded.append(node.pos)
        for a in actions(maze, node.pos):
            s = result(node.pos, a)
            child = Node(s, node, a, node.cost + 1)
            if maze[s] == "G":
                path = child.get_path_from_root()
                return {"path": [n.pos for n in path], "actions": [n.action for n in path[1:]],
                        "path_cost": child.cost, "expanded": expanded,
                        "frontier": [n.pos for n in frontier], "reached": reached}
            if s not in reached:
                reached.add(s)
                frontier.append(child)
    return {"path": None, "actions": None, "path_cost": None, "expanded": expanded,
            "frontier": [], "reached": reached}
```

```python
def mark_solution(maze, res):
    """Bản sao mê cung: '.' ô đã mở rộng, 'F' frontier, 'P' đường đi (bản đồ gốc không bị sửa)."""
    m = np.copy(maze)
    for p in res["expanded"]:
        if m[p] == " ":
            m[p] = "."
    for p in res["frontier"]:
        if m[p] in " .":
            m[p] = "F"
    for p in res["path"] or []:
        if m[p] not in "SG":
            m[p] = "P"
    return m
```

```python
for file in ["small_maze.txt", "large_maze.txt", "open_maze.txt", "empty_maze.txt"]:
    with open(file, "r") as f:
        m = mh.parse_maze(f.read())
    res = bfs(m)
    print(f"{file}: path length = {res['path_cost']}, nodes expanded = {len(res['expanded'])}")
    mh.show_maze(mark_solution(m, res))
```

| Mê cung | Độ dài đường đi | Số nút mở rộng |
|---|---|---|
| small | 19 | 90 |
| large | 210 | 618 |
| open | 54 | 679 |
| empty | 14 | 92 |

---

## 5. Câu hỏi thảo luận (mục 4.3 của đề)

**Câu 6. Khi nào BFS dùng nhiều bộ nhớ hơn A\*? Vì sao heuristic tốt làm giảm vùng trạng thái phải khảo sát?**

BFS loang đều theo mọi hướng, nên bộ nhớ tỉ lệ với mọi trạng thái trong bán kính $d$ quanh S. A\* chỉ mở rộng các nút có $f(n) = g(n) + h(n) < C^*$ (và một phần các nút có $f = C^*$). BFS tốn bộ nhớ hơn hẳn khi có **vùng trống hoặc nhiều nhánh mở ra các hướng không dẫn tới đích**: open maze BFS giữ 683 nút, A\* 275; L maze 152 so với 69. Heuristic càng sát $h^*$, điều kiện $g(n) + h(n) < C^*$ càng chặt, và tập nút thỏa điều kiện càng gần với chính đường đi tối ưu. Với $h = h^*$, A\* chỉ mở rộng các nút trên đường tối ưu. Ngược lại, ở mê cung chỉ một lời giải (large), mọi ô trên hành lang duy nhất đều phải đi qua nên hai thuật toán gần như ngang nhau.

**Câu 7. Tại sao DFS có thể cho đường đi rất dài hoặc gặp vấn đề trong môi trường có chu trình?**

- DFS đi sâu theo thứ tự hành động cố định và trả về lời giải **đầu tiên** gặp được, không xét độ dài. Nếu nhánh đầu tiên đi vòng (L maze: 122 bước so với 16; empty: 80 so với 14), DFS vẫn chấp nhận.
- Khi không có `reached`, DFS không nhớ trạng thái đã thăm ở nhánh khác. Không kiểm tra chu trình thì nó đi vòng vô hạn (A → B → A → …). Có kiểm tra chu trình theo đường đi thì hết vòng vô hạn, nhưng trong vùng trống số đường đơn tăng theo hàm mũ, nên DFS quét lại cùng một vùng rất nhiều lần (Bài 7: gần 200 000 lần mở rộng lại trên open maze).

**Câu 8. Tại sao thứ tự sinh trạng thái con ảnh hưởng mạnh đến DFS nhưng ít ảnh hưởng hơn đến BFS?**

- DFS luôn đi theo con được lấy ra đầu tiên **cho tới tận cùng** rồi mới xét con khác. Thứ tự quyết định nhánh nào được thử trước, tức quyết định gần như toàn bộ hành vi: đường đi tìm được, số nút mở rộng, độ sâu. Notebook 02 cho thấy cùng L maze, DFS có thể cho 122 hoặc 26 bước tùy thứ tự hướng.
- BFS mở rộng **hết** các nút ở độ sâu $k$ trước khi sang độ sâu $k+1$. Thứ tự chỉ thay đổi thứ tự trong cùng một lớp, nên có thể đổi *đường đi tối ưu cụ thể* (khi có nhiều đường ngắn nhất) hoặc số nút mở rộng ở lớp cuối, nhưng **không đổi chi phí lời giải**.

**Câu 9. So sánh GBFS và A\*: thuật toán nào sử dụng $g(n)$, $h(n)$, $f(n)$? Ảnh hưởng đến tính tối ưu như thế nào?**

| | $g(n)$ | $h(n)$ | $f(n)$ | Tối ưu? |
|---|---|---|---|---|
| GBFS | Không dùng | Có | $f = h$ | Không |
| A\* | Có | Có | $f = g + h$ | Có, nếu $h$ chấp nhận được (tree search) / nhất quán (graph search) |

GBFS chỉ nhìn "còn bao xa tới đích" mà bỏ qua "đã đi bao xa", nên dễ bị heuristic dẫn vào đường vòng (medium: 152 so với 68). A\* cân bằng cả hai. Vì $f(n)$ là cận dưới của chi phí mọi lời giải đi qua $n$, A\* không thể chọn một lời giải đắt hơn $C^*$ trước khi xét hết các nút có $f < C^*$.

**Câu 10. Manhattan và Euclidean khác nhau ra sao trong lưới chỉ cho phép đi lên/xuống/trái/phải?**

Xem mục 2.4. Tóm tắt:
- Manhattan bằng đúng số bước khi không có tường ($h_M = h^*$ trên mê cung rỗng).
- Euclidean nhỏ hơn (thấp nhất khoảng $h_M / \sqrt{2}$ theo đường chéo) vì giả định đi được theo đường thẳng.
- Cả hai đều chấp nhận được và nhất quán, nhưng $h_E \le h_M \le h^*$, nên Manhattan trội hơn và A\* + Manhattan mở rộng ít nút hơn (rỗng: 14 so với 61; open: 211 so với 516).

**Câu 11. Weighted A\* thay đổi điều gì khi tăng trọng số heuristic?**

Với $f = g + w\,h$:
- $w = 0$: uniform-cost search (BFS khi chi phí bằng nhau).
- $w = 1$: A\*.
- $w \to \infty$: GBFS.

Tăng $w > 1$ làm thuật toán "tin" heuristic hơn. Thường số nút mở rộng giảm, nhưng heuristic không còn chấp nhận được, nên chỉ bảo đảm $\text{cost} \le w\,C^*$. Thực nghiệm ở medium: chi phí 68 → 102 ($w = 5$) → 152 ($w = 100$). Số nút không phải lúc nào cũng giảm: khi heuristic dẫn sai hướng (open maze), A\* trọng số phải mở lại nhiều nút (717 với $w = 100$ so với 211 của A\*).

**Câu 12. Cycle checking làm giảm lặp trạng thái nhưng có thể làm tăng chi phí quản lý dữ liệu như thế nào?**

Xem mục 3.5. Tóm tắt:
- Kiểm tra đường đi tốn $O(m)$ cho mỗi con.
- Kiểm tra frontier cần thêm bảng băm, cập nhật mỗi lần push/pop.
- Đưa lên đỉnh stack cần xóa phần tử giữa stack ($O(|frontier|)$, hoặc xóa lười nhưng để lại bản ghi chết).
- Muốn loại hết đường dư thừa thì phải lưu `reached`, khi đó mất lợi thế bộ nhớ $O(bm)$ của DFS.

---

## 6. Tóm tắt

| Thuật toán | Frontier / hàm ưu tiên | Đầy đủ | Tối ưu | Bộ nhớ | Ghi chú từ thực nghiệm |
|---|---|---|---|---|---|
| BFS | FIFO | Có | Có (chi phí bằng nhau) | $O(n)$ | Ổn định nhưng khảo sát rộng (open: 679 nút) |
| DFS | LIFO, không `reached` | Có (hữu hạn, có kiểm tra chu trình) | Không | $O(bm)$ | Cần PA3 để vượt vùng trống; đường đi có thể rất dài |
| IDS | DLS với giới hạn tăng dần | Có | Có (với PA1) | $O(bd)$ | Bùng nổ thời gian ở vùng trống; PA3 làm mất tối ưu |
| GBFS | $h(n)$ | Có (có `reached`) | Không | $O(n)$ | Nhanh nếu heuristic chỉ đúng hướng |
| A\* | $g(n) + h(n)$ | Có | Có ($h$ nhất quán) | $O(n)$ | Ít nút nhất trong các thuật toán tối ưu; hòa điểm quan trọng |
| Weighted A\* | $g(n) + w\,h(n)$ | Có | Không khi $w > 1$ | $O(n)$ | Chi phí ≤ $w\,C^*$; không phải lúc nào cũng nhanh hơn |

**Kết luận chính:**

1. **Heuristic** là yếu tố quyết định hiệu quả của A\*. Heuristic cần đúng với mô hình di chuyển (Manhattan cho lưới 4 hướng), càng sát $h^*$ càng tốt (landmark), và phải tạo được độ dốc rõ ràng cho $f$, nếu không thì phải xử lý hòa điểm cẩn thận.
2. **Cycle checking** cho DFS không có `reached` chỉ thật sự hiệu quả khi quản lý cả frontier (PA3). Nhưng chính cơ chế này lại làm IDS mất tính tối ưu: luôn có đánh đổi giữa bộ nhớ, thời gian và tính tối ưu.
3. Không có thuật toán tốt nhất cho mọi mê cung: hành lang hẹp (large) làm mọi thuật toán gần như ngang nhau, còn vùng trống (open, empty, L) làm lộ rõ điểm mạnh và yếu của từng thuật toán.
