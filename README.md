# CPU-Bound-Application



## Stress Test


CPU를 극단적으로 사용하는 프로그램은 I/O 연산을 적게하고, CPU 연산을 많이 사용한다.
CPU 연산을 많이 사용하기위해 Hash 연산, 그 중에서도 MD5 Hash 연산을 반복적으로 사용하는 애플리케이션으로 성능 부하 테스트를 해보려고한다.

<br>

## Artillery 

Artillery는 성능을 측정하고 시각적 자료로 결과를 보여준다.
artillery-scripts 폴더를 만들고 그 안에 cpu-test.yaml 파일을 생성한다.

### Artillery Script
```yaml
# Artillery 스트레스 테스트 스크립트 (cpu-test.yaml)
config:
# 내 서버의 외부 IP
  target: "http://..."
  phases:
# 60초 동안 성능을 측정하고 매초 1명의 새로운 vUser(virtual user)를 만든다.
    - duration: 60
      arrivalRate: 1
      name: Warm up
scenarios:
  # We define one scenario:
  - name: "just get hash"
    flow:
      - get:
          url: "/hash/123"
```
서버에서 애플리케이션이 실행되고 있는 상태에서 해당 스크립트를 실행시킨다.

<pre>
artillery run --output report.json cpu-test.yaml
</pre>

실행시키고나면 cpu-test.yaml 이 있는 폴더에 report.json 이라는 파일이 생긴 것을 확인할 수 있다. json 파일로는 확인이 힘드니까 html 페이지로 시각화하여 확인해보자.
<pre>
artillery report report.json
</pre>
###
### 최신 버전 Artillery 는 UI가 변경되어서 곡선 그래프를 확인할 수 없어요!

```
 npm install -g artillery@1.6.0 
```
위의 명령어로 Artillery 1.6.0 버전을 다운받아서 곡선 그래프를 확인하자.

만약 이미 Artillery 를 다운 받았다면
```
npm uninstall -g artillery
```
위의 명령어를 먼저 실행하여 이미 설치된 artillery를 지우고 다시 다운받으면 된다.

<br>

## Stress Test를 할 때 염두해야할 점



- 예상 TPS보다 여유롭게 성능 목표치를 잡는다. 예상 TPS가 1000정도라면 트래픽이 튀는 상황을 대비해 최소 3000 ~ 4000 이상으로 여유롭게 인스턴스를 구성해야한다.
- 일정한 레이턴시를 유지하도록 목표를 정해야한다. 처음에는 짧은시간, 적은 TPS로 시작 → TPS를 점점 증가시켜서 언제부터 짧은 시간조차 버티지 못하는지 확인한다.
- API에 기대하는 Latency를 만족할 때까지, 서비스의 Peak 시간에 유입되는 트래픽을 버틸 수 있는 수진이 될 때까지 성능을 테스트해봐야한다. 가장 먼저 단일 요청에 대한 Latency가 몇인지 확인해본다.
    - 단일 요청에 대한 Latency가 기대 Latency 보다 높다면 Scale-Out으로 해결되지 않는다. 이런 경우 해당 코드가 비효율적으로 작성되었거나, 해당 API에서 실행되는 I/O가 병목인 경우가 많다. 또는 네트워크에서 Latency가 발생했을 수도 있다. 다양한 방면에서 확인해보는 것이 중요하다.
- 짧은 시간 높은 트래픽을 견딜 수 있는 기준을 찾았다면 스트레스 테스트 시간을 아주 길게 테스트 해본다. 장기적으로 자원이 고갈되는 상태일 수도 있기 때문이다.

<br>
