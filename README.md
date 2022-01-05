# viewmodel
ViewModel은 구글이 개발자들을 위해 이 Clean Architecture를 쉽게 구현할 수 있도록 라이브러리들을 만들었는데 이를 Android Architecture Components (AAC)라고 부르며 그중 하나가 바로 ViewModel이다.

[절대 MVVM의 ViewModel과 AAC의 ViewModel은 다른 것이다.](https://github.com/tnvnfdla1214/MVVM-VM-AAC-VM-)

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

### Activity(Lifecycle Extensions)
외부에서 인자를 안넣어줄 필요가 없는경우 (파라미터가 없는경우)
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

## 필요한곳에서 인스턴스 만들어주기
ViewModel class 작성해준후 사용하려는 Activity나 Fragment가서 실제 객체 인스턴스를 만들어줘야하는데 그

방법이 엄청나게 많다 팩토리를 이용하네 마네 어쩌구 저쩌구 일단 간단하게 보면 두가지 유형으로 나눌수있다.

그냥 만들어주기 vs 생성자로 뭐 집어넣어서 view모델 생성하기 

후자의 경우 ViewModelFactory를 직접 만들어줘야한다.

### 외부에서 인자를 안넣어줄 필요가 없는경우 (파라미터가 없는경우)
#### 1. Lifecycle Extensions
이 방법은 가장 편한 방법 중 하나입니다. androidx.lifecycle의 lifecycle-extensions 모듈을 가져와 사용하면 됩니다.

먼저, module 수준의 build.gradle 에 다음과 같이 디펜던시를 추가해줍니다.

 ```Kotlin
dependencies {
    // ...
    implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
}
```
 ```Kotlin
class MainActivity : AppCompatActivity() {
 
    private lateinit var noParamViewModel: NoParamViewModel
 
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
 
        /* use ViewModelProvider's constructor provided from lifecycle-extensions package */
        noParamViewModel = ViewModelProvider(this).get(NoParamViewModel::class.java)
    }
}
```
noParamViewModel = ViewModelProvider(this).get(NoParamViewModel::class.java)

요부분이 초기화 해주는건데 this는 ViewModelOwner가 들어가는거니 Activity나 Fragment가 들어가는곳입니다.

Fragment에서 부모 액티비티 넣고싶으면 requireActivity()넣어주면 되겠지

#### 2.ViewModelProvider.NewInstanceFactory
한마디로 안드로이드에서 기본제공하는 팩토리를 명시해서 사용하는방법입니다.

이번에 살펴볼 방법은 NewInstanceFactory 입니다.

이는 안드로이드가 기본적으로 제공해주는 팩토리 클래스이며, ViewModelProvider.Factory 인터페이스를 구현하고 있습니다. 따라서 ViewModel 클래스가 파라미터를 필요로 하지 않거나, 특별히 팩토리를 커스텀 할 필요가 없는 상황에서는 1번 방법을 사용하거나, 2번 방법을 사용하면 되겠습니다.

2번 방법은 1번에서 추가해준 lifecycle-extenstions 모듈을 추가하지 않아도 사용가능한 방법입니다.
 ```Kotlin
class MainActivity : AppCompatActivity() {
 
    private lateinit var noParamViewModel: NoParamViewModel
 
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
 
        noParamViewModel = ViewModelProvider(this, ViewModelProvider.NewInstanceFactory())
            .get(NoParamViewModel::class.java)
    }
}
```
#### 3.Android KTX 사용
[Android KTX](https://developer.android.com/kotlin/ktx)는 다양한 코틀린 확장함수들을 모아놓은 라이브러리다.
 ```Kotlin
//[KTX] 라이브러리 추가 
implementation "androidx.fragment:fragment-ktx:1.3.4"
```
 ```Kotlin
 //Activity, Fragment 에서의 초기
private val mainViewModel by viewModels<MainViewModel>()
```
by viewModels 키워드를 사용하여 간편하게 초기화해 줄 수 있다. 위의 4가지 개념과 같은방법이다
 ```Kotlin
 //프래그먼트간의 데이터 공유를 위하여 액티비티에서 만들어주는경우
private val mainViewModel by activityViewModels<MainViewModel>()
```
부모의 액티비티에서 뷰모델만들어서 자식 프래그먼트들끼리 공유하는경우 
by activityViewModels 키워드를 사용해 초기화 해준다 -> ViewModelOwner에 requireActivcity()넣어서 부모에서 만들어 주는것과 같은것이다.
### 외부에서 인자를 넣어줘야 하는 경우 (파라미터가 있는 경우)
#### ViewModelProvider.Factory
파라미터가 없던 ViewModelFactory의 연잔선상에서 Factory를 구현하면 파라미터를 소유하고 있는 ViewModel 객체의 인스턴스를 생성할 수 있다.

직접 구현한 Factory 클래스에 파라미터를 넘겨주어 create() 내에서 인스턴스를 생성할 때 활용하면 된다.
 ```Kotlin
 //ViewModel
class HasParamViewModel(val param: String) : ViewModel()
```
 ```Kotlin
 //ViewModelFactory
class HasParamViewModelFactory(private val param: String) : ViewModelProvider.Factory {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return if (modelClass.isAssignableFrom(HasParamViewModel::class.java)) {
            HasParamViewModel(param) as T
        } else {
            throw IllegalArgumentException()
        }
    }
}
```
 ```Kotlin
 //Activity (or Fragment)
class MainActivity : AppCompatActivity() {
 
    private lateinit var hasParamViewModel: HasParamViewModel
 
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
 
        val sampleParam = "Ready Story"
 
        hasParamViewModel = ViewModelProvider(this, HasParamViewModelFactory(sampleParam))
            .get(HasParamViewModel::class.java)
    }
}
```
이렇게해서 외부에서 뭐넣어주려면 팩토리 만들어서 사용하면된다.
