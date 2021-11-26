---
title: 백준 10021번 Watering the Fields
layout: post
tags: [cpp]
---
### 문제 개요
> Due to a lack of rain, Farmer John wants to build an irrigation system to send water between his N fields (1 <= N <= 2000).  
> Each field i is described by a distinct point (xi, yi) in the 2D plane, with 0 <= xi, yi <= 1000. The cost of building a water pipe between two fields i and j is equal to the squared Euclidean distance between them:  
> (xi - xj)^2 + (yi - yj)^2
> FJ would like to build a minimum-cost system of pipes so that all of his fields are linked together -- so that water in any field can follow a sequence of pipes to reach any other field.  
> Unfortunately, the contractor who is helping FJ install his irrigation system refuses to install any pipe unless its cost (squared Euclidean length) is at least C (1 <= C <= 1,000,000).  
> Please help FJ compute the minimum amount he will need pay to connect all his fields with a network of pipes.  
> 
> Input  
> Line 1: The integers N and C.  
> Lines 2..1+N: Line i+1 contains the integers xi and yi.
> 
> Output  
> Line 1: The minimum cost of a network of pipes connecting the fields, or -1 if no such network can be built.

### 우선순위 큐와 최소 스패닝 트리
간단하다. 우선순위 큐를 이용해 최소 스패닝 트리를 만들되, 간선 정보에 대해 일정 조건(>=C)을 만족한 경우에만 우선순위 큐에 넣어줘서 연산하면 된다.
### 코드
```c++
#include <iostream>
#include <queue>
#include <vector>
using namespace std;

int N, C, cummul, cnt;//입력용 N과 C, 누적합 저장할 cummul, 연결된 간선 수를 지정할 cnt
int x, y;//간선 정보 입력에 쓸 x, y
int parent[2000];//각 정점의 부모정보
vector<pair<int,int>> v;//입력을 저장할 배열
priority_queue<pair<long long, pair<int,int>>> pq;//간선 정보를 받아줄 우선순위 큐

int mypow(int xx) {//제곱함수
    return xx * xx;//인자값의 제곱을 반환
}

int find(int x) {//조상의 조상까지 거슬러가는 함수
    if (parent[x] == x) return x;//부모가 자신과 같으면 반환
    else return find(parent[x]);//아니면 조상찾아 올라감
}

void unionxy(int x, int y) {//부모를 같게 만드는 함수
    x = find(x);//x의 부모를 찾음
    y = find(y);//y의 부모를 찾음
    parent[y] = x;//후자의 부모를 전자의 부모와 같게 함
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(NULL); cout.tie(NULL);

    cin >> N >> C;//입력
    for (int i = 0; i < N; i++) {//N회동안
        cin >> x >> y;//입력
        int vs = v.size();//현재 벡터의 사이즈를 변수로 받음
        for (int j = 0; j < i; j++) {//i(벡터)의 크기만큼
            long long aa = mypow(v[j].first - x) + mypow(v[j].second - y);//간선의 비용을 측정
            if (aa >= C) pq.push({ -aa, {j,i} });//간선 비용이 C보다 크면 우선순위 큐에 넣어준다. 오름차순으로 하기 위해 비용은 - 붙여줌
        }
        v.emplace_back(x, y);//벡터에 일단 입력값을 집어넣어 나중에 쓸 수 있게 한다. 이 과정에서 현재 벡터의 사이즈와 다음 i값이 일치하게 됨.
        parent[i] = i;//i의 부모를 i로 만들어준다
    }
    while (!pq.empty()) {//우선순위 큐가 빌 동안에
        long long num = -pq.top().first;//맨 위의 첫 인자는 음수를 붙여 넣었으므로 다시 양수로 만든다
        int start = pq.top().second.first, end = pq.top().second.second;//각각 간선의 두 노드를 나타낸다
        pq.pop();//맨 위를 날림
        if (find(start) == find(end)) continue;//부모가 같다면 이미 그래프에 연결되어 있으니 날린다
        unionxy(start, end);//위에서 중단되지 않았다면 부모를 같게 하고
        cnt++;//간선 수를 증가시킨다
        cummul += num;//총 비용에 해당 간선의 비용을 추가한다
        if (cnt == N - 1) {//어쨌거나 그래프에 붙어만 있으면 되기 때문에, 간선 수의 최솟값은 무조건 N - 1이므로, 이때 해당 그래프가 완성되었을 것이다
            cout << cummul;//그 때 cummul을 출력
            return 0;//함수 종료
        }
    }
    cout << -1;//우선순위 큐가 빌때까지 돌았다면 최소 스패닝 트리가 불가능하니 -1을 찍어준다
}
```
### 해설
크루스칼 알고리즘을 이용해, 부모가 같다면 트리에 산입된 걸로 생각하고 처리한다. 이 때, 우선순위 큐를 이용해 간선들을 정렬되게 하여 맨 위부터 차례로 넣어주는게 관건.
### 여담
간선 조건의 >= C인데 > C라고 해서 자꾸 틀렸다. 이상과 초과를 잘 구분하자. 
