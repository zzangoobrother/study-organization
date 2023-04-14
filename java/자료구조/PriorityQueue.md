# PriorityQueue
PriorityQueue는 Heap 자료구조 기반으로 되어 있습니다.

### Heap 자료구조
최소값과 최대값을 빠르게 찾기위한 완전 이진 트리를 사용합니다.
최소힙은 root 노드값이 제일 작으며, 부모노드는 자식 노드보다 값이 작음을 항상 만족합니다.
최대힙은 root 노드값이 제일 크며, 부모노드는 자식 노드보다 값이 큼을 항상 만족합니다.

### Heap 삽입
1. 가장 밑단의 노드에 삽입합니다.
![20230414](https://user-images.githubusercontent.com/42162127/232048269-7d158d84-4af8-47b4-aaf8-cf079499744a.PNG)

2. 부모노드와 비교해서 자식노드가 크다면 교환, 작다면 해당 부분에서 정지합니다.
![20230414](https://user-images.githubusercontent.com/42162127/232048434-1daa9218-dfc0-4880-b55e-66b308358862.PNG)

