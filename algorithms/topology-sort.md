## 1. 위상 정렬(Topology Sort)
- 선후 관계가 정의된 그래프구조에서 정렬을 하기 위해 사용한다.
  - 즉, 먼저 방문할 노드가 존재할 때 위상 정렬을 사용할 수 있다.
- 비순환 그래프(Directed Acyclic Graph)이어야 사용할 수 있다.
- Queue 자료구조를 사용하여 진입 차수(in-degree)가 0인 노드를 먼저 방문한다.
  - Queue에는 진입 차수가 0인 노드가 차례대로 저장된다.

## 2. 위상 정렬 코드
<img width="664" alt="스크린샷 2025-01-27 오후 12 36 04" src="https://github.com/user-attachments/assets/61c7d928-7a22-420d-bc80-65007c91aa7f" />

사진을 기반으로 간선 정보를 저장하였다.
```java
public class 위상정렬 {

    static final int SIZE = 8; // 노드 개수

    static int[] inDegree = new int[SIZE + 1]; // 각 노드의 진입 차수
    static List<List<Integer>> graph = new ArrayList<>(); // 간선 정보
    static Queue<Integer> que = new ArrayDeque<>(); // 진입 차수가 0인 노드 삽입
    static List<Integer> visited = new ArrayList<>(); // 방문 순서

    public static void main(String[] args) {
        for (int i = 0; i <= SIZE; i++) {
            graph.add(new ArrayList<>());
        }

        initGraph(); // 간선 설정
        initInDegree(); // 진입 차수 설정

        // 1. 진입 차수가 0인 노드 큐에 삽입
        for (int v = 1; v <= SIZE; v++) {
            if (inDegree[v] == 0) {
                que.offer(v);
            }
        }

        // 2. 모든 노드의 진입 차수가 0이 될 때까지 반복
        while (!que.isEmpty()) {
            // 3. 노드의 방문 순서 기록
            int v = que.poll();
            visited.add(v);

            // 4. 노드(v)와 연결된 간선 제거
            for (int node : graph.get(v)) {
                inDegree[node]--;
                if (inDegree[node] == 0) {
                    que.offer(node);
                }
            }
        }

        // 5. 위상 정렬을 수행하여 방문한 노드의 순서 출력
        for (int v : visited) {
            System.out.println("방문 노드 : " + v);
        }
    }

    static void initGraph() {
        graph.get(1).add(2);
        graph.get(1).add(4);

        graph.get(2).add(5);
        graph.get(2).add(6);

        graph.get(3).add(6);

        graph.get(4).add(2);
        graph.get(4).add(8);

        graph.get(7).add(3);

        graph.get(8).add(6);
    }

    static void initInDegree() {
        for (int i = 1; i <= SIZE; i++) {
            for (int v : graph.get(i)) {
                inDegree[v]++;
            }
        }
    }
}
```

## 3. 위상 정렬로 풀 수 있는 문제
- https://www.acmicpc.net/problem/1766
