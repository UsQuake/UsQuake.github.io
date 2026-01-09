---
layout: default
title: Java vs Kotlin
---

## Java랑 다른 점
### 변수
  * 모든 변수는 객체다
    - 기본적인 primitive 타입(Int, Long, Short, Double, Float, Boolean, Byte)도 클래스로 취급한다!
    
  * Mutable 지원
  
    - var -> 변수 val -> 상수
    
  * 늦은 초기화(lazy init) 지원
  
    - lateinit 키워드 붙이기
      ``` kotlin
      fun main()
      {
        lateinit var : String //뒤늦게 대입하므로 var타입
      }
      ```
      
    - primitive 타입(Int, Long, Short, Double, Float, Boolean, Byte)에는 적용이 불가능 하다
     
    - by lazy {} 꼴로 lazy초기화하는 클래스를 정의 가능
     
      ``` kotlin
      val lazyData: Int by lazy{
         println("in lazy....")
         10
      }
      fun main()
      {
        println("in main....")
        assert(lazyData + 10 == 20)
      }
      ```
    
### 모던한 타입/컬렉션 지원(Pair, Range, Null-Safe-Type ..etc)

   * **기본 컬렉션**
      - 기본적으로 ***Immutable*** 타입이다
      
      - mutable***컬렉션 종류*** 타입으로 명시해야만 ***Mutable***하게 쓸 수 있다
      
      - ***컬렉션 종류***Of()(내용물1, 내용물2,...etc)꼴로 한번에 자료형을 정의할 수 있다
      
      - **List**
      
        + 리스트 내용물이랑 인덱스를 같이 뽑아낼 수 있다
        
          ``` kotlin
          fun main()
          {
           var list = listOf(10,20,30)
           for((index, value) in list.withIndex())
           {
            assert((index + 1) * 10 == value)
            /*assert((0 + 1) * 10 == 10) */
            /*assert((1 + 1) * 10 == 20) */
            /* .......etc........ */
           }
          }
          ``` 
     - **Map** 
     
       + 순서도 있고, Iterable 하다
       
       + Pair타입이나 "key" to "value" 형태로 자료를 넣을 수 있다
       
         ```kotlin
         fun main()
         {
          var map = mapOf<String, String>(Pair("one", "hello"), "two" to "world")
          assert(map.get("one").equals("hello"))
          assert(map.get("two").equals("world"))
          assert(map.size == 2)
         }
         ```
        
     - **Set**

        + 순서도 있고, Iterable 하다
        
          ``` kotlin
          fun main()
          {
           var set = mutableSetOf(5,4,3,2,1)
           //set = {0,1,2,3,4,5}
           assert(set.size == 5)
           for((index, value) in set.withIndex())
           {
             assert((5 - index) == value)
           }
          }
          ``` 
        
  * **Pair**
  
    - (변수 A, 변수 B...)꼴로 치환 가능한 데이터 클래스
    
      ```kotlin
      fun main()
      {
       var A = Pair("sss",3)
       assert(A == Pair("sss",3))
       
       var (a,b) = Pair("sss",3)
       assert(a.equals("sss"))
       assert(b.equals(3))
      }
      ```
      
    - Map 컬렉션에서 사용 가능
    
  * **Range**(Rust랑 거의 비슷함)
  
    - A~B까지 -> A..B꼴로 for문에서 유용하게 사용 가능
    
      ```kotlin
      fun main(){
       val arr = arrayOf(0,1,2,3,4)
       /* arr = {0,1,2,3,4} */
       for(i in 0..4)
       {
        assert(arr[i] == i)
        /*assert(arr[0] == 0)*/
        /*assert(arr[1] == 1)*/
        /*assert(arr[2] == 2)*/
        /* .....etc.........*/
       }
      }
      ```
      
    - 변수/상수 in "구간" 꼴로 범위 안에 있는지 확인 가능 
    
      ```kotlin
      fun main(){ 
         assert(6 in 0..9)
      }
      ```
      
  * **Any**
  
    - 모든 타입(클래스)의 어머니격인 타입(클래스)
    
    - when문에서 타입을 유연하게 쓰도록 활용 가능
    
      ```kotlin
      fun main(){
         var data: Any = 10
         when(data){
          is String ->println("data is String Type!")
          20, 30 ->println("data is 20 or 30")
          in 1..10 ->println("data is in 1..10!")
          else ->println("data is Not Valid Type!")
         }
      }
      ```
      
  * **Unit**
  
    - c언어의 void 형과 비슷하다
    
    - 함수형 프로그래밍할 때 void 생기면 곤란한 상황을 대처한다
    
      ```kotlin
      fun some(): Unit{
       println(10 + 20)
      }
      fun main()
      {
       assert(some() == Unit)
      }
      ```  
      
  * **Null-Safty Type**
  
    - **?연산자**
    
      + Null을 사용할지 말지 명시적으로 표현하는 방식이다
      
        ``` kotlin
        fun main()
        {
         var data1: Int = 10 //Null 대입 불가
         var data2: Int? = null //Null 대입 가능
        }
        ``` 
        
    - **Nothing** 
    
      + Unit과는 차이가 있다 보통 예외 처리에 쓴다고 한다.
      
        ```kotlin
        fun some(): Nothing{
         throw Exception()
        }
        fun main()
        {
         some() //예외 발생!
        }
        ```
        
      + 이것도 ? 연산 사용가능하다! 
      
        ```kotlin
        fun some(): Nothing?{
         return null
        }
        fun main()
        {
         assert(some() == null)
        }
        ```
        
### OOP

  * 생성자
  
    - 주 생성자 + 멤버 선언 동시에
    
      ```kotlin
         class User(var name: String){
         }
      ```
      
    - 주 생성자 -> init 블럭에서 초기화
    
      ```kotlin
         class User(_name: String){
          var name: String
          init
          {
           this.name = _name
          }
         }
      ```
      
    - 주 생성자 생략 + 보조 생성자(constructor()) 사용 
     (보조 생성자 같은 걸 왜 만들었냐고 묻지 말자..자바가 원래 오버로딩 안 되서 그렇다..ㅠ)
     
      ```kotlin
         class User{
          var name: String
          constructor(_name: String)
          {
           this.name = _name
          }
         }
      ```
      
    - 주 생성자 + 보조 생성자(constructor()) 연쇄 호출 적용 
    
      ```kotlin
       class User(name: String){
       
        var count: Int = 0 //Initialize가 아니라 Re-Assign이므로 무조건 variable타입
        val name: String = name
        var email: String = "aaaa@gmail.com" //Initialize가 아니라 Re-Assign이므로 무조건 variable타입
        
        constructor(name: String, count: Int)
        : this(name) {
         this.count = count
        }
        
        constructor(name:String, count: Int, email: String)
        : this(name, count) {
        this.email = email
        }
       }
      ```
      
  * 상속 
  
    - 상속할 함수/변수/상수 앞에 open 키워드로 상속 표현
    - 상속 받는 함수/변수/상수는 override 키워드 사용
    - 상속 받은 함수에 대해 오버로딩 X 무조건 가상함수로 작성(언어 레벨에섬 막힘)
    
      ```kotlin
      open class Vehicle(open var name: String){
       open val maxPassengerCount: Int = 0 
      }
      class Plane(override var name: String) : Vehicle(name){
       override val maxPassengerCount: Int = 20
      }
      fun main()
      {
       var IncheonBusanLine: Vehicle = Plane("747") //Java 베이스이므로, 다형성이 기본임
       assert(IncheonBusanLine.maxPassengerCount == 20)
      }
      ```
      

    
    
  * 접근 제한자: 
  
    - public : 모든 파일(기본적으로 getter setter를 만드는 public이 기본 타입이다)
    - internal : 같은 파일
    - protected : 사용 불가
    - private : 파일 내부에서 이용 가능
   
      ```kotlin
      open class Super{
        var publicData = 10
        protected var protectedData = 20
        private var privateData = 30
      }
      class Sub: Super(){
         fun subFun() {
            publicData++
            protectedData++
            privateData++ // 오류
         }
      }
      fun main() {
           val obj = Super()
           obj.publicData++
           obj.protectedData++ // 오류
           obj.privateData++ // 오류
      }
      ```
      
  * Data Class
  
    - equals()비교시 객체Id가 아니라 멤버 내용물을 비교한다!
    
      ```kotlin
      data class DataClass(val name: String, val email: String, val age: Int)
      class NonDataClass(val name: String, val email: String, val age: Int)
      fun main() {
           val obj1 = NonDataClass("kkang", "a@a.com", 10)
           val obj2 = NonDataClass("kkang", "a@a.com", 10)
           assert(!obj1.equals(obj2))
           
           val data1 = DataClass("kkang", "a@a.com", 10)
           val data2 = DataClass("kkang", "a@a.com", 10)
           assert(data1.equals(data2))
      }
      ```
      
  * 컴파일러 자동 생성 멤버 함수
  
    - equals() : 객체id를 비교한다. (Data클래스는 객체의 lateinit 변수를 제외하고 내부 변수들의 내용을 비교한다)
    
      ```kotlin
      data class DataClass(val name: String, val email: String, val age: Int) {
      lateinit var address: String
      constructor(name: String, email: String, age: Int, address: String)
        : this(name, email, age) {
           this.address = address
      }
      fun main() {
           val obj1 = DataClass("kkang", "a@a.com", 10, "seoul")
           val obj2 = DataClass("kkang", "a@a.com", 10, "busan")
           assert(obj1.equals(obj2))
      }
      ```
    - toString() : 객체id를 String으로 포매팅한다. (Data클래스는 객체의 lateinit 변수를 제외하고 내부 변수들의 내용을 포매팅한다)
      
      ```kotlin
      data class DataClass(val name: String, val email: String, val age: Int)
      class NonDataClass(val name: String, val email: String, val age: Int)
      fun main() {
           val obj = NonDataClass("kkang", "a@a.com", 10)
           val data = DataClass("kkang", "a@a.com", 10)
           println("non data class toString : ${obj.toString()}")
           println("data class toString : ${data.toString()}")
      }
      ```
      
  * Object Class
  
    - 임시 클래스로 전역으로 사용 가능한 단일 오브젝트를 만들어준다.
    
      ```kotlin
      val obj = object{
       var data = 10
       fun some() {
        println("data : $data")
       }
      }
      fun main() {
           obj.data = 20 //오류
           obj.some() //오류
      }
      ```    
    - 상속도 가능하다.
    
      ```kotlin
      open class Super{
       open var data = 10
       open fun some() {
        println("i am super some() : $data")
       }
      }
      val obj = object: Super(){
       override var data = 20
       override fun some() {
        println("i am object some() : $data")
       }
      }
      fun main() {
           obj.data = 30
           obj.some() 
      }
      ```  
      
  * Companion Object Class
  
    - 클래스 내부에 object class를 만들어준다.
    
      ```kotlin
      class MyClass {
       companion object {
        var data = 10
        fun some() {
         println("data : $data")
        }
       }
      }
      fun main() {
           MyClass.data = 20
           MyClass.some()
      }
      ``` 
