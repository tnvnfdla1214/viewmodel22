# viewmodel
ViewModel은 구글이 개발자들을 위해 이 Clean Architecture를 쉽게 구현할 수 있도록 라이브러리들을 만들었는데 이를 Android Architecture Components (AAC)라고 부르며 그중 하나가 바로 ViewModel이다.

절대 MVVM의 ViewModel과 AAC의 ViewModel은 다른 것이다.

ViewModel은 **UI 관련 데이터를 저장하고 관리해주는 역할**이다. 예시를 들어보자.

<img src="https://user-images.githubusercontent.com/48902047/142818045-250734ac-2b67-466d-9488-88d24e7b7fc8.gif"></img>

스마트폰을 가로로 회전하니 텍스트가 초기 값으로 돌아가는 걸 알 수 있다.

이런 현상이 발생하는 이유는 바로 **생명 주기** 때문이다. 화면 회전이 이루어지면 액티비티가 Destroy 됐다가 다시 Create 되기 때문에 기존의 데이터가 날라가는 것이다. 예시를 회전할 때로 들었지만 이런 문제가 다양한 상황에서 발생한다.

 ```Kotlin
class MainActivity : AppCompatActivity() {
 
    override fun onCreate(savedInstanceState: Bundle?) { // 이놈이 바로
        super.onCreate(savedInstanceState) // 그놈이다.
        setContentView(R.layout.activity_main)
    }
}
```
기존에는 이러한 문제를 saveInstanceState를 통해 해결할 수 있었다.

우리가 액티비티를 생성하면 항상 자동생성되는 저놈이 바로 그놈이었던 것이다.

액티비티가 파괴되기 전 세이브하고 싶은 데이터를 저놈을 통해 onCreate로 넘겨주면 데이터를 날리지 않고 계속 이용할 수 있는 것이다. 하지만 이 방법에는 다음과 문제가 있다.

+ 담을 수 있는 데이터가 적다.
+ 담을 수 있는 데이터의 형태가 제한된다.
+ onCreate에서 작업을 처리해야 하므로 UI 컨트롤러가 해야 할 일이 늘어나며 화면을 띄우는데 시간이 오래 걸린다.

UI 컨트롤러(액티비티, 프래그먼트)에서 데이터를 관리하자니 생명 주기에 따라서 값이 사라지고..

saveInstsanceState로 해결하려고 했더니 문제가 많네..

UI 컨트롤러가 데이터에 관여하지 않도록 따로 떼어버릴 순 없을까?? 🤔

하여 이 문제를 해결하기 위해 고안된 것이 바로 ViewModel이다.

#### 사용하려는 이유

 ```Kotlin
class MyViewModel : ViewModel() {
    private val users: MutableLiveData<List<User>> by lazy {
        MutableLiveData().also {
            loadUsers()
        }
    }
 
    fun getUsers(): LiveData<List<User>> {
        return users
    }
 
    private fun loadUsers() {
        // Do an asynchronous operation to fetch users.
    }
}
```
ViewModel의 한 예시이다.

ViewModel을 상속받는 클래스를 만들어 데이터를 저장하고 관리하는 로직을 간단하게 구현하였다.

<img src="https://user-images.githubusercontent.com/48902047/142818929-9527e8b7-28ab-4254-aeac-c846b947500b.png"></img>

이렇게 생성된 ViewModel은 액티비티 혹은 프래그먼트와 다른 생명주기를 가지게 된다.

finish 메서드가 호출됐을 때 혹은 사용자가 직접 뒤로 가기 버튼을 눌러 액티비티를 종료했을 때 onCleared 메서드를 통해 ViewModel은 비로소 소멸이 된다. 즉, 생명 주기가 더 길다는 뜻이다. 

이렇게 뷰 모델을 분리하면 다음과 같은 장점이 있다.

+ 생명 주기에 영향을 받지 않고 데이터를 유지할 수 있다.
+ UI 컨트롤러와 데이터가 분리된다.
+ 프래그먼트 간의 데이터 공유가 쉬워진다.

## 사용법

### gradle 추가

 ```Kotlin
android {
    .
    .
    dataBinding {
        enabled true
    }
}
```

 ```Kotlin
dependencies {
    def lifecycle_version = "2.3.0"
    // ViewModel
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:$lifecycle_version"
    // LiveData
    implementation "androidx.lifecycle:lifecycle-livedata-ktx:$lifecycle_version"
}
```
모듈 수준의 gradle에는 위와 같이 추가해주면 된다.

LiveData는 뷰 모델과 단짝 친구처럼 붙어 다니는 라이브러리라고 보면 된다.

### Layout

 ```Kotlin
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">
    <data>
        <variable
            name="user"
            type="com.example.selfstudy_kotlin.UserViewModel" />
    </data>
 
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical"
        tools:context=".MainActivity">
 
        <TextView
            android:id="@+id/textView_height"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"/>
 
        <Button
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="키 증가"
            android:onClick="@{()->user.increase()}"/>
    </LinearLayout>
</layout>
```

우선 레이아웃 파일이다. 버튼을 클릭하면 증가하는 파일이다.

### ViewModel

 ```Kotlin
class UserViewModel(): ViewModel() {
    private var _height = MutableLiveData<Int>()
 
    val height: LiveData<Int>
        get() = _height
 
    init {
        _height.value = 170
    }
 
    fun increase() {
        _height.value = _height.value?.plus(1)
    }
}
```
ViewModel 파일을 생성하기 위해선 우선 상속을 받아야 한다.

ViewModel과 AndroidViewModel을 상속받을 수 있는데 후자에 대한 설명은 뒤에서 설명할 예정이다.

특별한 상황이 아니라면 ViewModel을 상속받으면 된다.

앞에서 설명했듯이 ViewModel은 보통 LiveData와 같이 쓰인다.

지금은 "아~ 화면에 보여줄 height이라는 데이터와 관리하는 로직이 뷰 모델 안에 있구나"정도만 알면 된다.

### Activity
 ```Kotlin
class MainActivity : AppCompatActivity() {
 
    private lateinit var binding: ActivityMainBinding
    private lateinit var userViewModel: UserViewModel
 
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        userViewModel = ViewModelProvider(this).get(UserViewModel::class.java)
        binding.user = userViewModel
 
        /*
        val nameObserver = Observer<Int> { it ->
            binding.textViewHeight.text = it.toString()
        }
 
        userViewModel.height.observe(this, nameObserver)
        */
 
        // 위에 주석 달은 걸 줄이면 이거임.
        userViewModel.height.observe(this, Observer {
            binding.textViewHeight.text = it.toString()
        })
    }
}
```
뷰 모델 인스턴스를 왠지 생성자를 통해 만들 것 같지만 ViewModelProvider을 이용해야 한다.

observe는 데이터의 변동 사항을 감지하는 건데.. Live Data와 관련된 이야기 이므로 패쓰

 ```Kotlin
ViewModelProvider(this).get(UserViewModel::class.java)
```
ViewModelProvider를 사용할 때 this를 넘겨주는 데 이는 owner를 의미한다.

ViewModelStore를 누가 소유하고 있느냐? -> this가 소유하고 있다 = MainActivity가 소유하고 있다.

그렇다면 ViewModelStore은 무엇일까?

ViewModelStore은 ViewModel 객체가 HashMap 구조로 저장되는 곳이다.

즉, get() 안에 있는 'UserViewModel..'은 객체를 찾아오기 위한 Key값으로 쓰이는 것이다.

<img src="https://user-images.githubusercontent.com/48902047/142988101-9767a25d-e101-4e55-8bb3-ee2ea4e97550.png"></img>

그림으로 설명하자면 이렇다.

뷰 모델을 HashMap 구조로 저장하니까 get() 메서드에 Key값을 넣어준 거고.

(만약 Key에 해당하는 Value가 없으면 생성하고 가져온다. 그래서 처음 뷰 모델 객체를 처음 만드는데도 set 따위가 아니라 get을 쓰는 것)

저 ViewModelStore를 소유하고 있는 주체가 MainActivity라는 것을 알려주는 것이다.

이를 통해 우리는 2가지 사실을 알 수 있다

+ 뷰 모델을 각각 다른 소유자가 생성하면 이는 별개의 메모리 공간을 사용하는 다른 객체가 된다.
+ 하나의 액티비티를 소유자로 지정해 사용하면 같은 ViewModel을 공유할 수 있다. = 데이터 공유 가능

 ```Kotlin
class MainActivity : AppCompatActivity() {
 
    private lateinit var binding: ActivityMainBinding
    private lateinit var userViewModel: UserViewModel // 이거랑
 
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        userViewModel = ViewModelProvider(this).get(UserViewModel::class.java) // 이거를
        binding.user = userViewModel
 
        userViewModel.height.observe(this, Observer {
            binding.textViewHeight.text = it.toString()
        })
    }
}
```
이 뷰 모델 객체를 생성하는 저 두 줄의 코드를
 ```Kotlin
class MainActivity : AppCompatActivity() {
 
    private lateinit var binding: ActivityMainBinding
    private val model: UserViewModel by viewModels() // 이 한 줄로 바꿀 수 있다.
 
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        binding.user = model
 
        model.height.observe(this, Observer {
            binding.textViewHeight.text = it.toString()
        })
    }
}
```
이렇게 한 줄로 대체할 수 있다. 

### 주의할 점

***뷰 모델은 Activity, Fragment, Context를 참조하면 안 된다.***

위에서 설명했듯이 뷰 모델은 액티비티나 프래그먼트보다 긴 생명주기를 가지고 있다.

만약 뷰 모델이 액티비티에 대한 참조를 가지고 있다고 했을 때

화면을 가로로 했다가 세로로 했다가 가로로 했다가 세로로 했다가.... x 100번을 했다고 해보자.

액티비티는 종료와 생성을 반복하겠지만 뷰 모델은 쭉 살아있기 때문에 이미 종료되어 사라진 액티비티의 참조를 그만큼 가지고 있을 것이다. 쓸데없는 것이 메모리를 차지하고 있는 현상, 즉 Memory Leak이 발생하기 때문에 참조를 하면 안 된다는 것이다.

단, applicationContext는 액티비티의 생명주기가 아닌 애플리케이션의 생명주기를 가지기 때문에 참조를 해도 괜찮다. 이 경우는 뷰 모델을 만들 때 ViewModel을 상속받는 것이 아니라 AndroidViewModel을 상속받으면 된다.

