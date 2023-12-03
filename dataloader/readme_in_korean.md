
##### Service_Caching
Recommend to caching items for each MEC server.

1. Performance Evaluation  
- 1.1 Description of datasets used
  - 1.1.1. MovieLens 25M Dataset 에서 2019년도 데이터만 추출 -> 1,202,602 from 25,000,095(10,619 users 41,440 movieIds)  
  - 1.1.2. 1월부터 11월까지 데이터셋 생성(12월은 데이터가 존재하지 않음) -> 1월의 경우 13,521 movieIds, 3,110 users(called *month_dataset*)  set start_timestamp, end_timestamp(e.g. 2019-01-01 00:00:00 , 2019-02-01 00:00:00)
  - 1.1.3. movieId remapping : 0부터 N-1으로 총 N개 movieId에 대해 Id 재할당 -> pareto distribution을 확인  
<p align="center"><img width="398" alt="image" align="center" src="https://github.com/Bae-hong-seob/Service_Caching/assets/49437396/ba6803cb-2327-412a-a18b-69351b5761dd"></p> <br>

  - 1.1.4. Service 의 종류는 K개가 있다고 가정. pareto distribution probability에 따른 상위 Y% 집단을 형성.<br>
    - 상위 Y%에서 K x Y 개의 그룹을 형성 -> n개의 movieId가 상위 Y% 집단을 형성한다고 하였을 때, n/(K x Y)개씩 순차적으로 묶음을 형성(총 K x Y개의 묶음 형성)<br>
    - 이후 K - (K x Y) 개의 묶음은 상위 Y% 집단을 제외한 나머지 movieId를 K - (K x Y) 등분하여 묶음을 형성(총 K - (K x Y)개 묶음 형성)<br>
    - 예를 들어 K=64, Y=0.2(20%)라고 할 때, pareto distribution probability를 내림차순으로 정렬하여 상위 20% 집단에 대해 12(64 x 0.2)개의 service mapping 진행(가장 높은확률부터 n/12개 movieId는 묶음 0번으로 할당) 묶음 0,1, ... , 11까지 형성. 이후 12개의 묶음에 사용된 movieId제외한 나머지 movieId에 대하여 52(64-12)등분하여 순차적으로 묶음을 형성.(묶음 12,13, ... , 63까지 형성)
   
  - 1.1.5. Mobility dataset : [divvy-tripdata](https://divvybikes.com/system-data) 의 23년7월 데이터 참고.(767,650 개의 데이터 존재)
    - 1.1.5.1. 'started_at' column 과 'ended_at' column의 차이를 이용하여 'time_gap'을 계산  
    - 1.1.5.2. 생성한 'time_gap'을 활용하여 time_interval(=6hours) 이하의 mobility data만 filtering
    - 1.1.5.3. 출발좌표 'start_lat'(x1), 'start_lng'(y1) 및 도착좌표 'end_lat'(x2), 'end_lng'(y2) 을 각 column별 min-max를 통해 mec가 존재하는 격자내[0,2]의 움직임으로 변환
    - 1.1.5.4. (x1,y1) (x2,y2) 사이의 Euclidean distance 및 theta 계산
<p align="center"><img width="398" alt="image" align="center" src="https://github.com/Bae-hong-seob/Service_Caching/assets/49437396/12892e2a-b3d2-4e68-a88d-b77f0d413dd3">
<img width="398" alt="image" align="center" src="https://github.com/Bae-hong-seob/Service_Caching/assets/49437396/f9e0d74f-cf9a-4416-9c5b-1187f480831b"></p> <br>
<p align="center"><img width="398" alt="image" align="center" src="https://github.com/Bae-hong-seob/Service_Caching/assets/49437396/b5f578d8-dfd9-4791-bc3e-5477d0038c39"></p> <br>

- 1.2. Experiment Environment
  - 9개의 MECs, 64개의 Service가 존재한다고 가정 -> M=9, K=64
  - M x M 공간에 U 명의 user를 random하게 배치(user 번호는 0부터 U-1까지 할당)
  - 각 MEC는 (x,y) 에 존재. (x=0 or 1 or 2 | y = 0 or 1 or 2)
  - 각 user는 Euclidean distance를 기반으로 가장 가까운 MEC를 통해 request.
  - time_interval : 6 hours. 6시간동안 reqeust한 목록을 바탕으로 heatmap 생성
  - *month_dataset*으로부터 *start_timestamp*부터 *end_timestamp*까지 *time_interval* (time t slot)마다 heatmap 생성
  - 총 (24hours/*time_interval*) x days x 11 months 개의 heatmap 생성.

- 1.3. Experiment Cases
  - 1.3.1. Case 1 : Homogeneous and time-invariant  
    - 1.3.1.1. 1.1.4에서 설정한 묶음 0번부터 차례대로 Service 0번으로 mapping.  
    - 1.3.1.2. 매 time slot마다 request 목록을 차례대로 user에게 mapping. if number of request > U, Add request list from the first user again. (e.g. U=1000, number of request(time t slot)=1002, user 0,1 request twice.)  
    - 1.3.1.3. 생성한 user 별 request list를 바탕으로 MEC 별 number of Service request 계산.  
    - 1.3.1.4. 각 MEC 별 number of Service request를 바탕으로 9 x 64(number of MECs x number of Service) 크기의 heatmap 생성
    - 1.3.1.5. Service popularity가 모든 MEC에서 동일한 경우를 가정.

  - 1.3.2. Case 2 : Heterogeneous-I (only space variant)
    - 1.3.2.1. 1.1.4에서 설정한 묶음 64개를 MEC 개수만큼 shuffling하여 앞쪽에 위치한 묶음부터 차례대로 Service로 mapping
    <img width="713" alt="image" src="https://github.com/Bae-hong-seob/Service_Caching/assets/49437396/dfae1c32-af91-487d-b0fa-abce4d9b4636">

    - 1.3.2.2. 매 time slot마다 request 목록을 차례대로 user에게 mapping. if number of request > U, Add request list from the first user again. (e.g. U=1000, number of request(time t slot)=1002, user 0,1 request twice.)
    - 1.3.2.3. 생성한 user 별 request list를 바탕으로 배정된 MEC를 고려하여 number of Service request 계산
    - 1.3.2.4. 각 MEC 별 number of Service request를 바탕으로 9 x 64(number of MECs x number of Service) 크기의 heatmap 생성
    - 1.3.2.5. Service popularity가 MEC별로 다른 경우를 가정.

  - 1.3.3. Case 3 : Heterogeneous-II (space time variant): the service popularity is different in each MECS and changes over time
    - 1.3.3.1. 매 time slot마다 1.1.4에서 설정한 묶음 64개를 MEC 개수만큼 shuffling하여 앞쪽에 위치한 묶음부터 차례대로 Service로 mapping
    - 1.3.3.2. 매 time slot마다 request 목록을 차례대로 user에게 mapping. if number of request > U, Add request list from the first user again. (e.g. U=1000, number of request(time t slot)=1002, user 0,1 request twice.)
    - 1.3.3.3. 생성한 user 별 request list를 바탕으로 배정된 MEC를 고려하여 number of Service request 계산
    - 1.3.3.4. 각 MEC 별 number of Service request를 바탕으로 9 x 64(number of MECs x number of Service) 크기의 heatmap 생성
    - 1.3.3.5. Service popularity가 매 time slot 마다 변경되며, MEC별로도 다른 경우를 가정.

  - 1.3.4. Case 4 : Heterogeneous-III (space time variant via user mobility)
    - 1.3.4.1. 1.1.4에서 설정한 묶음 0번부터 차례대로 Service 0번으로 mapping.
    - 1.3.4.2. 매 time slot마다 request 목록을 차례대로 user에게 mapping. if number of request > U, Add request list from the first user again. (e.g. U=1000, number of request(time t slot)=1002, user 0,1 request twice.)
    - 1.3.4.3. 매 time slot마다 user mobility data에서 random 추출을 통해 theta 및 distance를 이용하여 현재 위치에서 이동진행. if user goes out of range, user is reallocated within range (e.g. user position (-0.3, 2.3) to (1.7, 0.3)). 이것은 유저가 mec 영역을 벗어날 경우 유출된 유저라고 가정하며 새로운 지역에서 새로 유입된 유저라고 가정한다. (mec 범위 내 user수는 동일하게 유지)
    - 1.3.4.4. 이동한 user의 위치에 따라 MEC 재배정
    - 1.3.4.5. 생성한 user 별 request list를 바탕으로 재배정된 MEC를 고려하여 number of Service request 계산
    - 1.3.4.6. 각 MEC 별 number of Service request를 바탕으로 9 x 64(number of MECs x number of Service) 크기의 heatmap 생성
    - 1.3.4.7. Service popularity가 MEC별로 다르며 user가 움직임에 따라 MEC별 service popularity 또한 time slot이 지남에 따라 다른 경우를 가정.
  
• Simulation Setup: (1) system parameters, (2) model parameters (3) model training 12(data, method)  
• Performance Metrics: cache hit rate, service delay, model train and inference time, etc.  
• variables: cache size, intensity of requests, user distribution, etc.  
