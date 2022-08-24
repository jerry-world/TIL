화면단에서 20000건의 데이터를 BE 단에 전달하여, 이를 DB에 upsert하는 로직에서 20000건의 데이터를
한번에 BE단에서 저장하려고하니 Request time out이 발생하여, 해결방법을 모색하던 중, 스크립트 단에서
time-out이 나지 않을정도로 나눠서 ajax call을 하면 되지 않을까 생각이 들었다.

그래서, Javascript를 잘하지 못하기 떄문에, 여러가지 리서치가 필요했고 이를 정리해보고자한다.

### Array를 n개씩 나누는 방법
사실 애당초 Array에 쌓을 때, n개씩 나눠 쌓으면 되지만, 1차원적인 접근으로 ajax call을 할 때, 
20000건이 들어있는 요청데이터를 2000개씩 총 10번 나눠서 보내보자로 접근했다.
여하튼 잘 몰랐던 거니깐 정리는 해놓자.

방법은 사실 간단하게 접근했다. 20000건을 특정 변수에 담아놓고, 20000건씩 짤라서 요청을 보내자.
요청을 보낼때마다 2000건씩 보내면 언젠간 0이 되겠지.

```javascript
let splitNum = 2000;

let originArray; //20000건의 데이터를 담고있음

let partitionSize = originArray/splitNum+(originArray%splitNum>0);

for (let i = 0; i < partitionSize; i++) {
    partitionArray.add(partitionSize.splice(0,splitNum));
    let partitionArray;
    
    $.ajax({
        //...
    });
    
}
```

그런데 내가 너무 단순하게 생각했다. 이게 콜백이 되기도전에 그다음 반복문이 되니깐, 원하는대로 splice되지 않아,
값이 0~2000, 2000~4000 이렇게 들어가야되는데, 0~2000, 0~2000 계속 이런식으로만 들어가는 것이다.

아.. 맞아.. javascript에서 ajax는 비동기로 처리하니깐 다음 for문 올것을 안기다리고 막 보내지...

그래서 찾아본게 async 옵션이다. ajax를 사용할때 옵션으로 async가 자동으로 활성화 되어있는데,
async를 비활성화하면 될 것이라고 생각했다.

```javascript
let splitNum = 2000;

let originArray; //20000건의 데이터를 담고있음

let partitionSize = originArray/splitNum+(originArray%splitNum>0);

for (let i = 0; i < partitionSize; i++) {
    partitionArray.add(partitionSize.splice(0,splitNum));
    let partitionArray;
    
    $.ajax({
        async: false,
        //...
    });
    
}
```

우선 머리속으로 생각하는 수준의 정도는 이정도인데, 뭐 이것보단 기존에 Array에 담을때 애당초 2000개씩 나눠담는 것으로 진행해야겠다.

(내일 테스트를 진행하고 다시 이곳에 업데이트 해야지.)

갑자기 드는 생각이... 굳이 쪼개지말고 타임아웃을 길게주는 것도 좋은방법일듯?...
쪼개서 넘기면 로그도 여러번 남고, 코드 수정도 많아지고 오히려 가독성이 저하될것같음...
timeout을 길게 주는 이유를 주석으로 남기고 평균치를 계산해서 좀더 넉넉하게 주는 방법으로 진행해보자.
