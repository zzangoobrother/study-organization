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

3. 다시 부모노드와 비교, 크다면 교환, 작다면 해당 부분에서 정지합니다.
![202304141](https://user-images.githubusercontent.com/42162127/232048739-6eb4f970-f70e-4ee6-b513-93c94a363d5b.PNG)

4. 다시 큰 값을 가진 자식과 위치를 교환 합니다. 최대힙 구조가 유지되면서 완성입니다.
![202304142](https://user-images.githubusercontent.com/42162127/232049183-d67c379e-a879-4841-aecf-f50b87bfad47.PNG)

