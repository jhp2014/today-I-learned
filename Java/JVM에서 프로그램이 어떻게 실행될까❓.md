


### ✅JVM Stack vs OS Process Stack
JVM Stack과 운영체제를 공부하면서 배운 Process의 Stack 영역을 비교해보자.
![](Pasted%20image%2020231119160519.png)
**유사한 부분** <sup>내 생각</sup> 
**1️⃣**두 경우 모두 method 호출 시 하나의 덩어리가 Stack에 쌓이게 된다. JVM은 Stack Frame으로, Process는 ebp와 esp로 method덩어리를 구분한다.
**2️⃣**또한, Method의 Local Variable을 할당 하기 위해 stack을 사용하고 있다는 점은 동일하다.

**차이점** <sup>내 생각</sup> 
**1️⃣**Process는 임시 결과 값을 CPU의 Register에 기록했지만, JVM은 오퍼랜드 스택에 기록한다.  



### 📄 참고자료
+ https://dzone.com/articles/introduction-to-java-bytecode
