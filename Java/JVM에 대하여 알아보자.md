
✅ JVM의 Main Job
+ load class files and execute the bytecodes they(JVM) contain
+ It contains `a class loader` , which loads class files from both the program and the Java API
+ The bytecodes are executed in an `execution engine`, which is one part of the virtual machine

JVM 내부에는 primordial class loader가 있고 얘가 load하는 건 신뢰할만하다. 주로 Java API의 Class file을 로드한다. 
하지만 다른 클래스 로더(이것은 Java 코드로 구현되어있고 다른 `.class` 파일과 동일하다.)가 load하는건 신뢰받지 못한다. 우리가 만든 클래스로더는 여러 개가 존재할 수 있다. 즉, 클래스를 런타임에 로드할 수 있고, 다양한 방식으로 로드할 수 있다.
클래스가 로드되면 JVM은 어떤 클래스 로더가 로드했는지 추적하고 있다.

로드 되어있는 Class A가 Class B를 참조하면 JVM은 Class A를 로드한 클래스 로더에게 Class B를 로드하라고 한다.
classes can by default only see other classes that were loaded by the same class loader
This is how Javaís architecture enables you to create multiple name-spaces inside a single Java application.
Classes loaded by different class loaders are in different name-spaces and cannot gain access to each other unless the application explicitly allows it.

클래스를 load한다는 말이 무엇일까? `.class` 파일은 자바 바이트 코드 파일이다. 이것이 실행 되기 위해서는 JVM으로 load 되어야 한다. 이 작업이 클래스를 load한다는 말이다.


When the virtual machine loads a class file, it parses information about a type from the binary data contained in the class file.
`.class` 가 load되면, 여기서 타입에 대한 정보를 분리한다.
It places this type information into the method area
그리고 method 영역에 이 정보를 위치시킨다. 

The methods of class ClassLoader allow Java applications to access the virtual machineís class loading machinery
클래스 로더 사용하면, (class loader object) java의 클래스 로딩에 접근가능

Also, for every type a Java Virtual Machine loads, it creates an instance of class java.lang.Class to represent that type
로딩되면 → `Class` Object 만들고 → 힙 영역에 존재한다.
Data for loaded types resides in the method area
타입에 대한 정보는 method 영역에 존재한다.


클래스 로더 하는 일
verify the correctness
allocate and initialize memory
assist in the resolution of symbolic references

로딩
Linking
1. 검증
2. 준비
3. Resolution :: transforming symbolic references from the type into direct references.
Initialization 초기화


but it must recognize class files.
JVM은 반드시 Class file은 인식해야한다.

primordial class loader 동작
fully qualified type name 있으면
an environment variable named CLASSPATH를 활용해서 찾는다.

동작 예시
if the primordial class loader is searching for class java.lang.Object, it will look for Object.class in the java\lang subdirectory of each CLASSPATH directory.

Memory for class (static) variables declared in the class is also taken from the method area.
static 변수도 method area에 존재한다.

The method area can also be garbage collected. Because Java programs can be dynamically extended via class loader objects, classes can become "unreferenced" by the application
오 method 영역도 가비지 컬렉터의 대상이 될 수 있네?

method에 load된 클래스 정보에는 constant pool이 존재한다.
A constant pool is an ordered set of constants used by the type including literals (string, integer, and floating point constants) and symbolic references to types, fields, and methods.
symbolic references가 잘 이해가 안가네. Chapter 6에서 다시 나온다고 한다. 

constant pool plays a central role in the dynamic linking of Java programs.

Constants (class variables declared final) are not treated in the same way as non-final class variables.
Every type that uses a final class variable gets a copy of the constant value in its own constant pool.
As part of the constant pool, final class variables are stored in the method area--just like non-final class variables.
But whereas non-final class variables are stored as part of the data for the type that declares them
오! 클래스 변수(아마 스태틱인듯)는 미리 메모리에 할당 되어있어야 한다.
하지만, 스태틱 파이널 변수는 다른 방식으로 취급된다.
둘다 method area에 저장 되는데, final은 constant pool의 일부이고 non-fianl은 그 타입을 선언한 데이터의 일부로 저장된다.
아직 뭔소린지 잘 모르겠다. 뒤에 더 설명한다고 한다.

Class Loader information is stored as part of the typeís data in the method area.

The virtual machine must in some way associate a reference to the Class instance for a type with the typeís data in the method area.
method의 Class 정보에 Class 인스턴스를 참조하는 것이 있어야 한다.

Your Java programs can obtain and use references to Class objects. One static method in class `Class`, allows you to get a reference to the Class instance for any loaded class
Class의 static method를 사용하면, 나는 Class인스턴스의 참조를 얻을 수 있다.
`forname()` 사용 시, 해당 Class가 현재 name space로 로드 될 수 있어야 한다.(이미 로드되어있으면 괜찮)

An alternative way to get a Class reference is to invoke getClass() on any object reference
또 다른 Class참조를 얻는 방법은 `인스턴스.getClass()`호출

Given a reference to a Class object, you can find out information about the type by invoking methods declared in class Class. If you look at these methods, you will quickly realize that class Class gives the running application access to the information stored in the method area.
Class 인스턴스를 참조하면, method 영역에 있는 Class정보를 얻을 수 있다.


method table
In addition to the raw type information listed above, implementations may include other data structures that speed up access to the raw data
JVM could generate a method table and include it as part of the class information it stores in the method area
위에 기본적인 정보 말고 method area에 있는 Class 정보에 빠르게 접근 할 수 있도록 method table을 만들 수 있다(JVM 구현체가 결정하는 듯하다.)

A method table is an array of direct references to all the instance methods that may be invoked on a class instance, including instance methods inherited from superclasses.
메소드 테이블은 인스턴스 메소드를 참조하고 있다. 따라서, 추상 클래스나, 인터페이스에서는 필요없다. 왜냐면 인스턴스화 되지 않으니까
A method table allows a virtual machine to quickly locate an instance method invoked on an object
인스턴스 메소드 접근 속도 올림

인스턴스가 처음에 할당되면 field는 기본값으로 초기화 되네.. 심지어 값을 제공했어도
private int a = 5 해도 처음에 a 는 0(int 기본 값) 이다.
그 다음에 내가 지정한 값으로 초기화 한다.


heap에 있는 객체 표현
instance variables / all its superclasses
In addition, there must be some way to access an objectís class data (stored in the method area) given a reference to the object
method 영역의 class정보도 포인팅 해야한다.


Java 객체에는 모두 lock이 존재한다.

Once a thread owns a lock, it can request the same lock again multiple times, but then has to release the lock the same number of times before it is made available to other threads

The pc register is one word in size, so it can hold both a native pointer and a returnValue.

스택 프레임의 로컬 변수의 첫번째 인덱스가 this일 수있다.(스태틱 메소드면 this 필요없다.)
You canít directly access a classís instance variables from a class method, because there is no instance associated with the method invocation. 
왜 static 메소드가 인스턴스 로컬 변수에 접근 못하는지. static method는 this가 없다. 


어라? 생각해보니까 constant pool은 method area에 존재하네?
Frame Date에 그 해답이 존재하는 것 같다. 
Java stack frame includes data to support constant pool resolution, normal method return, and exception dispatch

Many instructions in the Java Virtual Machineís instruction set refer to entries in the constant pool
명령어가 constant pool을 많이 참조한다.
Some instructions merely push constant values of type int, long, float, double, or String from the constant pool onto the operand stack. Some instructions use constant pool entries to refer to classes or arrays to instantiate, fields to access, or methods to invoke.
단순히 상수를 가지고 오던, 메소드 또는 객체를 참조하던..

it uses the frame dataís pointer to the constant pool to access that information
프레임 데이터가 상수 풀 참조 포인터를 가지고 있는듯하다. 

the frame data must assist the virtual machine in processing a normal or abrupt method completion
프레임 데이터가 

The frame data must also contain some kind of reference to the methodís exception table, which the virtual machine uses to process any exceptions thrown during the course of execution of the method
프레임데이터는 예외 테이블 보유
예외 테이블에는 아래와 같은 정보 존
an index into the constant pool that gives the exception class being caught,
and a starting position of the catch clauseís code.

The virtual machine uses the information in the frame data to restore the invoking methodís frame
frame data를 통해 함수 호출 복구 즉, 예외 발생 시 처리한다는 거
또한, 디버깅을 위한 정보도 가지고 있을 수 있다.

Like references to fields and methods of other classes, references to the fields and methods of the same class are initially symbolic and must be resolved before they are used.
같은 클래스의 메서드 호출도 심볼릭 레퍼런스 사용한다.


. It then pops the double and int parameters (88.88 and one) from addAndPrint()ís operand stack and places them into addTwoType()ís local variable slots one and zero
함수에서 함수 호출 과정이다. 매개변수값을 오퍼랜드 스택에 쌓아놓고 스택 프레임 받으면 기존 스택프레임의 값을 빼다가 새 스택프레임의 로컬 변수 자리에 값 넣어준다.

When addTwoTypes() returns, it first pushes the double return value (in this case, 89.88) onto its operand stack. The virtual machine uses the information in the frame data to locate the stack frame of the invoking method, addAndPrint()
함수 종료 전 return 값을 OP stack에 넣고 기존 함수를 frame data를 이용해 찾아서 기존 함수의 op stack에 넣어고 종료할 함수의 스택프레임을 제거한다.
값 전달하는걸 메모리가 겹치게 해놔도 되는구나?

자바 쓰레드 우선순위 설정 가능 
우선순위 설정하면 native thread의 우선순위랑 연결된다.


There is no way to put more than one class or interface into a single class file.

The first entry in the list has an index of one, the second has an index of two, and so on. Although there is no entry in the constant_pool list that has an index of zero, the missing zeroeth entry is included in the constant_pool_count
상수 풀 인덱스는 1부터 시작하지만,  상수 풀 갯수는 +1이 된다. 즉 실제론 1~14여도 개수는 15가 될것이다.