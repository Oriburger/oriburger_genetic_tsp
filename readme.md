# TSP Genetic Algorithm by Oriburger

---

## 1. 개요

본 프로젝트는 1000개의 2차원 좌표로 이루어진 도시 정점을 입력으로 받아 
모든 도시를 순회하는 가장 작은 total cost를 찾는 TSP(Traveling Salesman Problem)을 해결하는 소스코드이다.
아래는 세부 문제 상세이다.
- 도시의 좌표는 최소 (0.0f, 0.0f), 최대 (100.0, 100.0f)이다.
- 도시의 초기 간선은 주어지지 않는다.

## 2. 모집단 구현 상세

1. 영역 분할
- 20.0f 단위로 도시를 두부처럼 분할한다.
- ```TreeRouteFinder```의 생성자와 ```initAreaAdjList()``` 함수 내에서 영역 데이터를 초기화
- ```initAreaAdjList()``` 함수는 각 정점과 가까운 최대 ```MAX_BRANCH_COUNT```개의 간선을 잇는다.

2. BackTracking 
- ```TreeRouteFinder::findAreaMinimumRoute``` 함수에서 진행
- 세부 파라미터는 아래와 같다.
```
void TreeRouteFinder::findAreaMinimumRoute
(
const int areaId         //현재 영역 ID
, int curr                //탐색할 정점
, vector<Node> currState  //지금까지 탐색한 route 정보
, double currCost         //지금까지 탐색한 route의 totalCost
){...}
```

3. Branch & Bound (Pruning)
- ```TreeRouteFinder::findAreaMinimumRoute```
- 영역 분할 단계에서 최대 branch 개수를 ```MAX_BRANCH_COUNT```로 한정 시켜놓음
- 또한 ```currCost```가 현재까지 찾은 ```minCost```보다 크다면, pruning을 진행
- ```
  void TreeRouteFinder::findAreaMinimumRoute(const int areaId, int curr, vector<Node> currState, double currCost)
  {
    ...
    
    for(auto &nextInfo : adj[curr])
    {
        const int next = nextInfo.second;
        const int cost = nextInfo.first;

        if(visited[next]) continue;
        if(currCost + cost > minCost[areaId]) break; //Pruning : 현재까지 찾은 minCost보다 크다면 가지를 침

        ..
    }
  }
  ```
- 휴리스틱
- ```
   const int tufuOrder[25] = {8, 3, 4, 9, 14
                              , 13, 18, 19,24, 23
                              , 22, 17, 16,21,20
                              , 15, 10, 5, 0, 1
                              , 2, 7, 6, 11, 12};
  ```
- 영역간 순회 순서를 강제하여 초기 모집단 fitness을 낮추는 것을 꾀함

## 3. Convex Hull Insertion
0. 추가 예정

## 4. GA구현 상세
0. 기본
- 염색체 표현 : 도시의 순열. 즉, 실제 경로를 표현
- 따라서 유전자는 각 도시가 됨.

1. 부모 선택
- 순위 기반 선택, 상위 20개 집단을 선택
- fitness를 기반으로 오름차순 정렬한 다음, 상위 20개를 제외하고 제거

2. 교차 연산
- 상위 15개 + 랜덤한 idx, 총 15 번의 crossover 진행
- 순열 교차 ```Chromosome GeneticSearch::crossover(const Chromosome& p1, const Chromosome& p2)```
- 이후 모집단에 push

3. 돌연변이 연산
- 범위 기반 돌연변이
- 상위 15개를 기반으로 10% 확률로 돌연변이 진행
- 최대 2%의 범위를 Shuffle

3. 적합도 선택
- 각 염색체의 실제 totalCost를 유클리드 거리 기반으로 산출
