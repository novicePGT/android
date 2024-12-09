# Data Class 의 Eqauls 와 HashCode
- Data Class 는 왜 사용하는 알아보고 Eqauls 와 HashCode 를 왜 override 해야하는지 알아보기

## Data Class 의 사용 이유?
- equals() 와 hashCode() 를 제공
- toString() 함수 제공
- copy() 함수 제공
- ComponentN
</br>
</br>

### 간단하게 알아보는 CompoentN
```kotlin
data class UserInfo(
  val userName: String,
  val userAge: Int
)

val (name, age) = UserInfo("이름", 20)
textViewName.text = name
textViewAge.text = age
```
- Data Class 인 UserInfo 를 풀어서 쓸 수 있다
- 순서대로 지정한 변수 명을 그대로 사용 가능하다
</br>
</br>

### 간단하게 알아보는 copy()
```kotlin
data class UserInfo(
  val userName: String,
  val userAge: Int
  val userId: Long
)

val firstUserInfo = UserInfo("이름", 20, 112)
val secondUserInfo = firstUserInfo.copy(userName = "두번쨰 이름", userId = 113)
```
- immutable 정의에서 유용하게 사용 가능
- copy를 통해 특정한 변수의 값만 변경한 객체 생성이 가능

### 간단하게 알아보는 eqauls() 와 hashCode()
![스크린샷 2024-12-09 오후 1 59 36](https://github.com/user-attachments/assets/6d1da40c-8c43-4e85-b81b-a850df0a1997)
```kotlin
class SomethingResponse(
    val name: String?,
    val age: Int
) {

    override fun hashCode(): Int {
        return ((name?.hashCode() ?: 0) * 31 + age) * 31
    }

    override fun equals(other: Any?): Boolean {
        return if (this !== other) {
            if (other is SomethingResponse) {
                val (name, age) = other
                return Intrinsics.areEqual(name, age)
            }
            false
        } else {
            true
        }
    }
}
```
- equals() 와 hashCode() 정의 규칙
  - class 에 eqauls() 를 재정의 했다면, hashCode 도 재정의 해야한다
  - 2개의 객체에 equals 가 동일하다면 '그 객체의 주소' hashCode 도 동일해야한다
  - 이런 조건을 지키지 않았을 경우 HashMap, HashSet 등 컬렉션 사용 시 문제가 발생한다
</br>
</br>

- 하지만 Data Class 를 활용하면 kotlin 에서 자동으로 eqauls 와 hashCode 를 정의해주기 떄문에 손대지 않는 편이 좋다

</br>
</br>

### 최종 결론: Data Class 를 활용한 eqauls 와 hashCode   
```kotlin
fun stringMemory() {
    val somethingItemOne = SomethingResponse(
        name = "name",
        age = 50,
        isSelected = false
    )
    
    val somethingItemTwo = SomethingResponse(
        name = "name",
        age = 50,
        isSelected = false
    )
    
    println("somethingItemOne == somethingItemTwo ${somethingItemOne == somethingItemTwo}")
    println("somethingItemOne === somethingItemTwo ${somethingItemOne === somethingItemTwo}")
    println("somethingItemOne hashCode ${somethingItemOne.hashCode()}")
    println("somethingItemTwo hashCode ${somethingItemTwo.hashCode()}")
}
```
- 위 2 객체는 동일한 값을 가지기 때문에 eqauls는 true 이다.
- 2 객체는 서로 해시코드가 달라야하지만 kotlin data class 의 eqauls 와 hashCode 정의로 인해 hashCode 도 동일한 값을 가진다
