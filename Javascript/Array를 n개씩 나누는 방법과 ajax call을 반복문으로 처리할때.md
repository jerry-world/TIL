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


---
무슨일이든 생각처럼 안되는 법이지.

우선 타임아웃은 불가능하였다. ajax call의 타임아웃을 커스텀하더라도, 뒷단 nginx나 api gateway에서 이미 막혀버린다.

그리고 asnyc 옵션도 순차적으로 보내주기만할 뿐 동기적으로 처리할 수는 없었다.

결국 타임아웃을 늘려 처리하는 방법은 불가능 하였고, 콜백을 받아 재귀하여 처리하는 방법을 생각했다.

동기처리를 위해서, 콜백에 재귀를 담아 처리하는 방식으로 진행했다.

### Callback 시, 재귀 호출
```javascript
let action = function(data, index){
    $.ajax({
        url: url, 
        success: function(){
            if(data.list.length<=index) {
                action(data, index++);
            }else{
                finish();
            }
        }
        //...
    });
}
```

물론 이렇게하고나서 다 완료되지는 않았다.. ㅋㅋㅋ 이것 말고도 BE쪽에서 다른 부하가 같이 물려있었다.

그래도 어쨌든 재귀를 활용하는 생각을 키웠다는 관점에서 꽤 공부가 된 것 같다.