# 안드로이드 코드 팁

## 깃허브 토큰 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

## Android Studio


<details>
<summary>ViewBinding, DataBinding</summary>

### ViewBinding


## java

```java

Activity

public ActivityMainBinding binding;

@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        setContentView(binding.getRoot());
    }


Fragment

public FragmentMylistBinding binding;

@Nullable
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        binding = FragmentMylistBinding.inflate(inflater);
//        return super.onCreateView(inflater, container, savedInstanceState);
        return binding.getRoot();
    }


Adapter

RecyclerPrditemBinding binding;

 @NonNull
    @Override
    public RecyclerView.ViewHolder onCreateViewHolder(@NonNull ViewGroup viewGroup, int i) {
        binding = RecyclerPrditemBinding.inflate(LayoutInflater.from(viewGroup.getContext()), viewGroup, false);
        return new likeViewHolder(binding);
    }


  class likeViewHolder extends RecyclerView.ViewHolder {

        RecyclerPrditemBinding binding;

        public likeViewHolder(@NonNull RecyclerPrditemBinding binding) {
            super(binding.getRoot());
            this.binding = binding;
        }
    }



@Override
    public void onBindViewHolder(@NonNull RecyclerView.ViewHolder viewHolder, @SuppressLint("RecyclerView") int position) {

        if (viewHolder instanceof categoryViewHolder) {

            /** viewBinding 이후에 onBindViewHolder에서 UI관련 작업할 때 유의사항 */

            binding = ((categoryViewHolder) viewHolder).binding; // 이 작업을 해주거나
            ((likeViewHolder) viewHolder).binding.getRoot().setBackgroundColor(context.getResources().getColor(R.color.category_select)); // 캐스팅을 한번 해줘야 잘 작동된다..

            

            }
        }

```

## kotlin

``` java

Activity 

private lateinit var binding: ActivityMainBinding

override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        /** viewBinding*/
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding!!.root)

        Init()
    }

Fragment

    lateinit var binding: FragmentPrdlistBinding

 override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {

        binding = DataBindingUtil.inflate(inflater, R.layout.fragment_prdlist, container, false)

  return binding.root
    }


Adapter

 inner class MyViewHolder(private val binding: RecyclerMyitemBinding) :
        RecyclerView.ViewHolder(binding.root) {
        fun bind(mylike: MyLike) {
            binding.mylike = mylike

            Glide.with(context).load(currentList[position].PRODUCT_IMAGE)
                .placeholder(R.mipmap.ic_launcher).error(R.mipmap.ic_launcher)
                .diskCacheStrategy(DiskCacheStrategy.ALL).centerCrop().into(binding.img)

            myLikeRepository = context?.let { MyLikeRepository(it) }!!
            myViewModel = myLikeRepository?.let { MyViewModel.getInstance(myLikeRepository) }!!
        }

        init {
            binding.imgLike.setOnClickListener(View.OnClickListener { view ->
//                binding.root.setOnClickListener(View.OnClickListener { view ->
                L.d(" click : $position")
                CustomToast(position = position)
            })
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MyViewHolder {
//        return MyViewHolder(_root_ide_package_.com.ojt.sampleappkotlin.databinding.RecyclerMyitemBinding.inflate(LayoutInflater.from(parent.context),parent,false))
        val binding =
            RecyclerMyitemBinding.inflate(LayoutInflater.from(parent.context), parent, false)
        context = parent.context
        return MyViewHolder(binding)
    }

```
## DataBinding

### kotlin

```java

/** dataBinding*/

Activity

private lateinit var binding: ActivityMainBinding
lateinit var topLabelViewModel: ToplabelViewModel

    private fun Init() {
        binding = DataBindingUtil.setContentView(this, R.layout.activity_main)
        topLabelViewModel = ViewModelProvider(this).get(ToplabelViewModel::class.java)

        binding.topLabelViewModel = topLabelViewModel
        binding.lifecycleOwner = this
    }

class ToplabelViewModel : ViewModel() {

    protected val TAG = this.javaClass.simpleName

    val _titleTxt = MutableLiveData<String>()   //MutableLiveData : 값의 get/set 모두를 할 수 있다.
    val titleTxt : LiveData<String> get()=_titleTxt //LiveData : 값의 get()만을 할 수 있다.

    init {
        _titleTxt.setValue("상품 리스트")
    }

    fun TitleChange(str: String) {
        _titleTxt.setValue(str)
        //_titleTxt.value=str
        //MutableLiveData를 setValue 해주면 UI의 적용 됨.
        //setValue or value 는 동기처리 postValue()는 비동기처리
    }
}

Layout

최상단을 layout으로 감싼 후 데이터 선언

<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

<data>

<import type="android.view.View" />
<import type="android.text.TextUtils" />

<variable
    name="topLabelViewModel"
    type="com.ojt.sampleappkotlin.viewmodel.ToplabelViewModel" />

<variable
    name="activity"
    type="com.danawa.estimate.kotlin.ui.activity.WriteActivity" />      //Activity도 등록하면 함수를 편하게 사용 가능하다.

</data>


<TextView
        android:id="@+id/txtTopLabel"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@{topLabelViewModel.titleTxt}"
        android:textSize="30dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />


   <ImageView
            android:id="@+id/imgSearch"
            android:layout_width="20dp"
            android:layout_height="20dp"
            android:layout_marginEnd="15dp"
            android:src="@drawable/ic_search"
            android:visibility="@{topLabelViewModel.titleTxt.equalsIgnoreCase(`즐겨찾기`)==true?View.VISIBLE:View.GONE,default=gone}"
            app:layout_constraintBottom_toBottomOf="@+id/txtTopLabel"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintTop_toTopOf="@+id/txtTopLabel" />


</layout>

RecyclerView
//Adapter 선언 후 @BindingAdapter 를 사용하면 xml에서 binding해줄 수 있다.

object MyBindingAdapter {
    @BindingAdapter("categoryItems")
    @JvmStatic
    fun categorySetItem(
        recyclerView: RecyclerView,
        categoryList: LiveData<ArrayList<Category>>
    ) {
        if (recyclerView.adapter == null) {
            val adapter = CategoryAdapter()
            recyclerView.adapter = adapter
        }

        categoryList?.let {
            val myAdapter = recyclerView.adapter as CategoryAdapter
//            Log.d("categorySetItem", it.value.toString())
            Log.d("categorySetItem", it.value?.size.toString())
            myAdapter.submitList(it.value)
            myAdapter.notifyDataSetChanged()
        }
    }

    @BindingAdapter("prdItems")
    @JvmStatic
    fun prdSetItems(recyclerView: RecyclerView, prdList: LiveData<ArrayList<Products>>) {
        if (recyclerView.adapter == null) {
            val adapter = ProductAdapter()
            recyclerView.adapter = adapter
        }

        prdList?.let {
            val myAdapter = recyclerView.adapter as ProductAdapter
//            Log.d("prdSetItems", it.value.toString())
            Log.d("prdSetItems", it.value?.size.toString())
            myAdapter.submitList(it.value)
            myAdapter.notifyDataSetChanged()
        }
    }


    @BindingAdapter("myItems")
    @JvmStatic
    fun mySetItem(recyclerView: RecyclerView, myList: LiveData<ArrayList<MyLike>>) {
        if (recyclerView.adapter == null) {
            val adapter = MyAdapter()
            recyclerView.adapter = adapter
        }

        myList?.let {
            val myAdapter = recyclerView.adapter as MyAdapter
//            Log.d("mySetItem", it.value.toString())
            Log.d("mySetItem", it.value?.size.toString())
            myAdapter.submitList(it.value)
            myAdapter.notifyDataSetChanged()
        }
    }


}




/** dataBinding*/

```

</details>

<details>
<summary>Android DeepLink </summary>

### 앱링크

1. Android Manifest추가
   
```java

    <activity>
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />
        <!-- <data android:scheme="http" android:host="zzzwww3.dothome.co.kr" /> -->
        <!-- <data android:scheme="https" android:host="zzzwww3.dothome.co.kr" /> -->
        <data android:scheme="test_scheme" android:host="zzzwww3.dothome.co.kr" />
        // scheme에 http나 https로 해야 앱링크... 아직 성공한 적 없음 
    </intent-filter>
    </activity>
```

2. 서버 root 경로에 assetlinks.json 파일 추가

        [
        {
        "relation": ["delegate_permission/common.handle_all_urls"],
        "target": {
            "namespace": "android_app",
            "package_name": "com.sktelecom.Danawa.Main.admin",
            "sha256_cert_fingerprints": ["B8:44:D9:CD:3F:AA:BD:A9:88:45:68:55:E6:A1:62:37:F3:9B:C9:CD:3B:F2:40:74:9F:1D:07:75:B8:D2:AD:5E"]
                    }       
        }
        ]

    sha256 확인방법 
    1. https://singo112ok.tistory.com/49
    2. Android Studio Build -> Generate Signed Bundle or APK -> Keystore 생성 후 빌드 후 생성된 key.keystore 경로 이동 후 cmd에 keytool -list -v -keystore my-release-key.keystore

3. 웹 사이트 주소가 아닌 ```<a href="test_scheme://zzzwww3.dothome.co.kr"></a>```로 해야 작동함.


4. 안드로이드 11의 "패키지 공개 상태 변경사항 정리

    https://tech.buzzvil.com/blog/tech-blog-package-visibility-in-android-11/

</details>


<details>
<summary>Kotlin </summary>


### 코틀린의 모든 타입은 기본적으로 널이 될 수 없는 타입이다.


1. 세미콜론을 붙이지 않아도 된다.
2. 변수 선언시 파스칼, 카멜 표기법을 권장한다.
   
        파스칼 표기법 : ClassName
        카멜 표기법 : className

3. 변수 선언 방법

        var : 일반적으로 통용되는 변수. 언제든지 읽기 쓰기가 가능함.
        val : 선언시에만 초기화 가능. 중간에 값을 변경할 수 없음.
        ex) runtime시 변경되지 말아야 할 값은 안전하게 val로 선언하는 것이 좋다.
        변수는 선언 위치에 따라 두가지 이름으로 부른다.
        클래스에 선언된 변수 : Property(속성)
        이 외의 Scope 내에 선언된 변수 : Local Variable (로컬변수)

4. 자료형

```kotlin
fun main() {
    var intValue:Int = 1234
    var longValue:Long = 1234L //L을 붙여 더 큰 메모리를 사용하는 정수임을 표시
    var intValueByHex:Int = 0x1af //16진수
    var intValueByBin:Int = 0b //2진수 (binary 약자)
    //코틀린은 8진수 표기는 지원하지 않는다.
    
    var doubleValue:Double = 123.5 //실수는 소수점을 포함해 숫자를 쓰거나
    var doubleValueWithExp:Double = 123.5e10 //필요시 지수 표기법을 추가한다.
    var floatValue:Float = 123.5f //Float는 f를 붙인다.

    var charValue:Char = 'a'
    var koreanCharValue:Char = '가'
    var stringValue = "one line string test"
    var multiLineStringValue = """multiline
    string
    test"""
    
    var booleanValue:Boolean = true


    var a: Int = 54321
    var b: Long = a.toLong() //int를 long형으로 변환


    var intArr = arrayOf(1,2,3,4,5) //int 배열
    var nullArr = arrayOfNulls<Int>(5) //비어있는 5크기의 배열
    
    int Arr[2] = 8 //index2에 8값 할당

    var a = 1234 //int
    var b = 1234L //long
    
    var c = 12.45 //Double
    var d = 12.45f //float
    
    var e = 0xABCD //16진수
    var f = 0b0101010 //2진수
    
    var g = true //boolean
    var h = 'C' //Char

}

```

5. 함수

```kotlin

/** a,b,c를 더해서 나오는 값이 Int이므로 fun add(a: Int, b: Int, c:Int): Int 맨 마지막 반환형 Int 선언.
반환값이 없다면 생략해도 된다. */

fun main() {
	println(add(5, 6, 7))
}
 
fun add(a: Int, b: Int, c:Int): Int {
    return a + b + c
    //리턴 발생 시 함수 중간이더라도 함수 종료
}


```



###  물음표(?)의 사용

    코틀린은 널이 될 수 있는 타입을 명시적으로 지원한다.
    타입 옆에 물음표(?)를 표시한다.
    null이 들어올 수 있는 경우에 붙여줌
    var notNull:Int = null	//오류
    var notNull:Int? = null	//정상

### 느낌표 두개(!!)의 사용

    null값이 절대 들어오면 안되는 경우에 붙여줌
    NULL이 될 수 있는 타입의 변수이지만, 현재 NULL이 아님을 주장할 수 있다.
    느낌표 2개(!!)를 변수 뒤에 붙인다.
    이 표시를 통해 null이 될 수 없는 변수에 null이 될 수 있는 타입을 주입할 수 있다.
    var notNull:Int = 0		//기본값은 null허용x
    var okNull:Int? = 10	//null이 들어올 수 있음을 의미
    notNull = okNull!!		//ofNull은 null을 허용한 상태이기 때문에 !!로 처리해야 오류없이 실행됨
    //그러나 위의 코드에서 okNull에 null값이 들어가면 오류가 발생함

### ex)
    // var a:Int = null -> error
    var a:Int? = null 
    var b:Int? = 10
    // var c:Int = b  -> error 
    var c:Int = b!!

### 안전한 호출 연산자 ?.

    (?.)은 null 검사와 메서드 호출을 한 번의 연산으로 수행한다.
    foo?.bar()
    foo가 null이면 bar() 메서드 호출이 무시되고 null이 결과 값이 된다.
    foo가 null이 아니면 bar() 메서드를 정상 실행하고 결과값을 얻어온다.

### 엘비스 연산자 ?:

    null 대신 사용할 디폴트 값을 지정할 때 편리한 연산자
    fun foo(s: String?) {
        val t: String = s ?: ""
    }
    s가 null이면 ""(빈 문자열)을 t에 넣고
    s가 null이 아니면 t에 s를 넣는다.

### 안전한 캐스트 as?

    자바 타입 캐스트와 마찬가지로 대상 값을 as로 지정한 타입으로 바꿀 수 없다면 ClassCastException이 발생한다.
    그래서 자바에서는 보통 is를 통해 미리 as로 변환 가능한 타입인지 검사해 봄
    as? 연산자는 어떤 값을 지정한 타입으로 캐스트하고, 변환할 수 없으면 null을 반환한다.
    foo as? Type
    foo is Type이면 foo는 Type으로 변환하고
    foo !is Type이면 null을 반환한다.


### let 함수

    let 함수를 안전한 호출 연산자와 함께 사용하면 원하는 식을 평가해서 결과가 널인지 검사한 다음에 그 결과를 변수에 넣는 작업을 간단한 식을 사용해 한꺼번에 처리할 수 있다.

    fun sendEmailTo(email: String) {
        println("Sending email to $email")
    }

    fun main(args: Array<String>) {
        var email: String? = "yole@example.com"
    //    sendEmailTo(email) -> error
        email?.let { sendEmailTo(it) }
    }


### open 

    자바에서는 클래스에 final이 붙지 않으면 모두 다른 클래스에서 상속이 가능합니다.
    하지만 코틀린에서의 클래스와 메서드는 기본적으로 final입니다.
    따라서 어떤 클래스의 상속을 허용하려면 해당 클래스 앞에 open 변경자를 붙여야 합니다. 그와 더불어 오버라이드를 허용하고 싶은 메서드나 프로퍼티의 앞에도 open 변경자를 붙여야 합니다.

```kotlin 
abstract class Animal {

    // 추상 메서드는 반드시 override 해야 함
    abstract fun bark()

    // 이 메서드는 하위 클래스에서 선택적으로 override 할 수 있다. (하거나 안하거나 자유)
    open fun running() {
        println("animal running!")
    }
}

class Dog() : Animal() {

    override fun bark() {
        println("멍멍")
    }

    // 이 메서드는 override 하거나 하지 않거나 자유.
    override fun running() {
        println("dog's running!")
    }
}    
```

### 나중에 초기화할 프로퍼티

    코틀린에서 클래스 안의 널이 될 수 없는 프로퍼티를 생성자 안에서 초기화하지 않고 특별한 메서드 안에서 초기화할 수 없다. 
    코틀린에서는 일반적으로 생성자에서 모든 프로퍼티를 초기화해야 한다.
    게다가 프로퍼티 타입이 널이 될 수 없는 타입이라면 반드시 널이 아닌 값으로 그 포로퍼티를 초기화해야 한다. 
    그런 초기화 값을 제공할 수 없으면 널이 될 수 있는 타입을 사용할 수 밖에 없는데, 이러면 모든 접근에 대해 널 검사를 넣거나 !! 연산자를 써야한다.
    
    lateinit 변경자: 프로퍼티를 나중에 초기화
    나중에 초기화하는 프로퍼티는 항상 var여야 한다. 
    val 프로퍼티는 final 필드로 컴파일되며, 생성자 안에서 반드시 초기화해야 한다.

    널이 될 수 있는 타입 확장: isNullOrEmpty(), isNullOrBlank

    타입 파라미터의 널 가능성
    타입 파라미터 T를 클래스나 함수 안에서 타입 이름으로 사용하면 이름 끝에 물음표가 없더라도 T가 널이 될 수 있는 타입이다. 
    t가 널이 될 수 있으므로 안전한 호출(?.)을 써야만 한다.


### LiveData?

    LiveData는 RxJava와 같은 observable 형태로 사용하며, 안드로이드 Lifecycle에 따라 데이터를 관리한다.
    Activity, Fragment의 라이프 사이클을 따르기에 활동에 대한 처리를 알아서 관리해 준다.
    구글 문서를 참고하면 아래와 같은 장점을 확인할 수 있다.
    - LiveData는 observable 패턴을 사용하기에 데이터의 변화를 구독한 곳으로 통지하고, 업데이트한다.
    - 메모리 누수 없는 사용을 보장한다.
    - Lifecycle에 따라 LiveData의 이벤트를 제어한다.
    - 항상 최신 데이터를 유지한다.
    - 기기 회전이 일어나도 최신 데이터를 처리할 수 있도록 도와준다.(AAC-ViewModel과 함께 활용 시)
    - LiveData의 확장도 지원한다.

    MutableLiveData와 LiveData의 구분?
    MutableLiveData와 LiveData를 구분해서 사용하는 이유를 알아보자.

        MutableLiveData : 값의 get/set 모두를 할 수 있다.
        LiveData : 값의 get()만을 할 수 있다.


<img src="https://velog.velcdn.com/images%2Fsoyoung-dev%2Fpost%2Fe80b71be-be26-453d-a505-b7d2ee02bd3d%2Fimage.png" width="500px" height="300px">

   

    build.gradle -> android { 

    /*  viewBinding, dataBinding 같이 적용 불가 **/    
    buildFeatures {viewBinding = true}
    buildFeatures {dataBinding = true}

    }


    dataBinding -> dataclass package명 앞에 소문자 사용할 것 -> 대문자 에러남
    MVVM 패턴 적용
</details>

<details>
<summary>Android Studio Plugins </summary>

        CodeGlane
        이 플러그인은 코드 편집기 우측에 전체 미니맵을 표시해준다. 코드 편집기 사용을 용이하게 만들어주고 간편한 페이지 이동을 지원해주는 플러그인이다.

        Mario Progress Bar
        이 플러그인은 Android Studio 내의 모든 Progress Bar를 Mario Progress Bar 모양으로 바꿔준다.
        지루한 빌드와 인스톨 시간동안 Mario를 보면서 힐링하도록 하자.
        
        Stickode
        자동완성 도와주는 플러그인
</details>


<details>
<summary>MultiCursor Keymap </summary>

    Clone Caret Above
    Clone Caret Below
    Select All Occurrences
    Add Selection For Next Occurrence
    Unselect Occurrence
    단축키 생성해주면 가능

</details>

<details>
<summary>AutoImport </summary>
    Settings(Ctrl + Alt + S) -> Editor -> General -> Auto Import -> Java (Add unambiguous imports on the fly 체크)
</details>

<details>
<summary> res</summary>



```xml

<!-- style.xml 일괄 적용 -->

<?xml version="1.0" encoding="utf-8"?>
<resources>

    <!-- Custom font style 일괄적용 -->
    <style name="TextViewStyle" parent="@android:style/Widget.DeviceDefault.TextView">
        <item name="android:fontFamily">@font/applesdgothicneom</item>
    </style>

    <style name="ButtonStyle" parent="@android:style/Widget.DeviceDefault.Button.Borderless">
        <item name="android:fontFamily">@font/applesdgothicneoeb</item>
        <item name="android:textAllCaps">false</item>
    </style>

    <style name="EditTextStyle" parent="@android:style/Widget.DeviceDefault.EditText">
        <item name="android:fontFamily">@font/applesdgothicneom</item>
        <item name="android:textCursorDrawable">@color/MainText</item>
        <item name="android:textColor">@color/MainText</item>
        <item name="android:textColorHint">@color/disable</item>

    </style>

    <style name="RadioButtonStyle" parent="@android:style/Widget.DeviceDefault.CompoundButton.RadioButton">
        <item name="android:fontFamily">@font/applesdgothicneom</item>

    </style>

    <style name="CheckboxStyle" parent="@android:style/Widget.DeviceDefault.CompoundButton.CheckBox">
        <item name="android:fontFamily">@font/applesdgothicneom</item>

    </style>


</resources>



<!-- themes.xml -->


v31 부터 자동으로 스플래쉬 화면이 생성된다 -> 방지하기 위한 코드

<style>
 <!-- 자동 스플래쉬화면 막기 (눈속임) -->
        <item name="android:windowSplashScreenBackground">@color/white</item>
        <item name="android:windowSplashScreenAnimationDuration">10</item>
        <item name="android:windowSplashScreenAnimatedIcon">@drawable/drawable_none</item> // drawable resource file을 drawable_none 이름으로 생성하여 비워두면 된다.
</style>


View 커스텀 할 때 (CustomView)

attrs.xml 에 추가

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <declare-styleable name="NewAttr">
        <attr name="strokeWidth" format="dimension" />
        <attr name="strokeColor" format="color" />
    </declare-styleable>
</resources>

xml 에 추가

  <CustomView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:strokeWidth="2dp"
        app:strokeColor="@android:color/black"/>

java 에 추가

public CustomView(@NonNull final Context context, @Nullable final AttributeSet attrs,
                  final int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initViews(context);
}


```

</details>


<details>
<summary>RecylcerView </summary>


```java 

// Aadapter 외부에서 ViewHolder 접근하기
((PrdAdapter.prdViewHolder) binding.prdRecycler.findViewHolderForAdapterPosition(position)).imgLike.setImageResource(R.drawable.star_on);
((ViewHolder) RecyclerView.findViewHolderForAdapterPosition(position)).접근하려는 View


//필터 적용한 리싸이클러뷰 만들기


public class LikeAdapter extends RecyclerView.Adapter implements Filterable {

    Context context;
    ArrayList<MyLike> likeList, filteredList; // 좋아요 리스트

    public LikeAdapter(ArrayList<MyLike> likeList, Context context) {
        this.likeList = likeList;
        this.filteredList = likeList;
        this.context = context;
    }

    @Override
    public Filter getFilter() {
        return new Filter() {
            @Override
            protected FilterResults performFiltering(CharSequence charSequence) {
                String charString = charSequence.toString();
                if (charString.isEmpty()) {
                    filteredList = likeList;
                } else {
                    ArrayList<MyLike> filteringList = new ArrayList<>();
                    for (MyLike name : likeList) {
                        if (name.getPRODUCT_TITLE().toLowerCase().contains(charString.toLowerCase())) { // charString이 좋아요 리스트의 상품 이름에 포함 될 때 
                            L.d("filter : " + charString.toLowerCase() + " // " + name.getPRODUCT_TITLE().toLowerCase());
                            filteringList.add(name);
                        }
                    }
                    filteredList = filteringList;
                }
                FilterResults filterResults = new FilterResults();
                filterResults.values = filteredList;
                return filterResults;
            }

            @Override
            protected void publishResults(CharSequence charSequence, FilterResults filterResults) {
                filteredList = (ArrayList<MyLike>) filterResults.values;
                notifyDataSetChanged();
                /** 좋아요 리스트가 있고 검색 할 때만 진동*/
                if (likeList.size() != 0) {
                    new FragmentMy().likeItemCheck(filteredList.size(), true);
                }
            }
        };
    }

    @Override
    public int getItemCount() {
        return filteredList.size();
    }

}

//화면에 보일 때 애니메이션 효과

@Override
  public void onViewAttachedToWindow(@NonNull RecyclerView.ViewHolder holder) {
      holder.itemView.setAnimation(new AnimationUtils().loadAnimation(context, R.anim.recycler_anim)); // 원하는 애니메이션 적용
      super.onViewAttachedToWindow(holder);
  }

//add item 할 때 
likeList.add(position,addObject); // 리스트에 특정 index에 Object 추가
myRecycler.getAdapter().notifyItemInserted(position);
myRecycler.getAdapter().notifyItemRangeChanged(position, likeList.size()); //notifyItemRangeChanged()안하면 뒤쪽 애들의 position이 업데이트 되지 않아서 밀리면서 꼬임
myRecycler.smoothScrollToPosition(position);
    
//kotlin MVVM databinding -> adapter(ListAdapter)에서 add item 할 때
val addList=currentList.toMutableList() // currentList 는 readOnly 직접적으로 수정하면 에러남.
addList.add(position,addObject)
submitList(addList) // mutableData를 submitList 해주면 notify알아서 해줌
//recyclerView scroll이 멈춰 있어서 추가 후 scroll 되게 해주는 코드
Handler().postDelayed({ // 딜레이 안주면 position이 0일 때 스크롤이 안되는 이슈 발생
    MyFragment.binding.myRecycler.smoothScrollToPosition(position)
}, 100)



// remove item 할 때 인덱스 에러 방지코드

likeList.remove(tempRemove); // 리스트에서 Object 지우기
myRecycler.getAdapter().notifyItemRemoved(position); // 지워진 인덱스 값
myRecycler.getAdapter().notifyItemRangeChanged(position, likeList.size());  //notifyItemRangeChanged()안하면 뒤쪽 애들의 position이 업데이트 되지 않아서 밀리면서 꼬임

//kotlin MVVM databinding -> adapter(ListAdapter)에서 remove item 할 때
val removeList=currentList.toMutableList() // currentList 는 readOnly 직접적으로 수정하면 에러남.
removeList.remove(tempRemove)
submitList(removeList) // mutableData를 submitList 해주면 notify알아서 해줌


// RecylcerView 모든 아이템이 그려진 후 

          ViewTreeObserver observer = recyclerView.getViewTreeObserver();
                        observer.addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
                            @Override
                            public void onGlobalLayout() {
                                // 모든 아이템이 그려진 후의 시점에서 실행됩니다.

                                new Handler().postDelayed(new Runnable() {
                                    @Override
                                    public void run() {
                                        progressBar.setVisibility(View.GONE);
                                    }
                                }, 0);
                                // 이벤트 리스너를 해제합니다.
                                observer.removeOnGlobalLayoutListener(this);
                            }
                        });


// 리싸이클러뷰 화면 높이만큼 미리 생성 후 캐싱처리

    //   리싸이클러뷰 init 붙여줄때 추가해줄 것
      LinearLayoutManager lm = new LinearLayoutManager(getActivity()) {
            @Override
            protected int getExtraLayoutSpace(RecyclerView.State state) {
                return getResources().getDisplayMetrics().heightPixels;
            }
        };
        recyclerMain.setLayoutManager(lm);
        recyclerMain.setItemViewCacheSize(20);


// recyclerview 첫 아이템, 마지막 아이템만 패딩 줄때 recyclerview 자체에 top,bottom padding 주고 android:clipToPadding="false" 하면 깔끔.


//리싸이클러뷰 선택한 아이템만 색상 다르게
    // 초기값    selectedPosition = -1 설정 
    public void onBindViewHolder(@NonNull RecyclerView.ViewHolder holder, int position) { 안에 사용할 것

    // 선택 항목 강조
    if(selectedPosition == position) { //선택 사항
        GradientDrawable shape = new GradientDrawable();
        shape.setCornerRadius(10);
        shape.setColor(Color.parseColor(clientOptionList.get("BG_COLOR")));
        shape.setStroke(2, Color.parseColor(clientOptionList.get("BOTTOM_COLOR")));
        holder.itemView.setBackground(shape);
        GradientDrawable Rshape = new GradientDrawable();
        Rshape.setCornerRadius(50);
        Rshape.setColor(Color.parseColor(clientOptionList.get("BOTTOM_COLOR")));
        ((BookViewHolder) holder).num.setBackground(Rshape);
        ((BookViewHolder) holder).title.setTextColor(Color.parseColor(clientOptionList.get("BOTTOM_COLOR")));
        ((BookViewHolder) holder).content.setTextColor(Color.parseColor(clientOptionList.get("BOTTOM_COLOR")));
    } else {  //선택 안된것
        GradientDrawable shape = new GradientDrawable();
        //shape.setCornerRadius(holder.itemView.getWidth()/2);
        shape.setCornerRadius(10);
        shape.setColor(Color.parseColor(clientOptionList.get("BG_COLOR")));
        shape.setStroke(2, Color.parseColor(grayBorderColor));
        holder.itemView.setBackground(shape);
        GradientDrawable Rshape = new GradientDrawable();
        //shape.setCornerRadius(holder.itemView.getWidth()/2);
        Rshape.setCornerRadius(50);
        Rshape.setColor(Color.parseColor(grayBorderColor));
        ((BookViewHolder) holder).num.setTextColor(Color.parseColor(clientOptionList.get("BG_COLOR")));
        ((BookViewHolder) holder).num.setBackground(Rshape);
        ((BookViewHolder) holder).title.setTextColor(Color.parseColor(grayTxtColor));
        ((BookViewHolder) holder).content.setTextColor(Color.parseColor(grayTxtColor));
    }


    holder.itemView.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            selectedPosition = position;
            notifyDataSetChanged();
        }
    });
    }



// RecyclerView 에서 좌우 스크롤 할때마다 Indicator 이미지 수정

    detailRecycler.setOnScrollChangeListener(new View.OnScrollChangeListener() {
    @Override
    public void onScrollChange(View v, int scrollX, int scrollY, int oldScrollX, int oldScrollY) {
        //int idx = (int) ((detailRecycler.getScrollX()- pageWidth / 3) / pageWidth + 1);
        //detailRecycler.offsetChildrenHorizontal()
        float pageWidth = detailRecycler.getWidth();
        int idx = (int) ((detailRecycler.computeHorizontalScrollOffset() - pageWidth / 3) / pageWidth + 1);
        int position = (int) idx;
        for (int i = 0; i < DetailActivity.circleIndicatorWrap.getChildCount(); i++) {
            if (i == position) {
                if (DetailActivity.circleIndicatorWrap.getChildAt(i) instanceof ImageView) {
                    ((ImageView) DetailActivity.circleIndicatorWrap.getChildAt(i)).setImageDrawable((changeDrawableColor(getApplicationContext(), R.drawable.selected_dot, MainActivity.clientOptionList.get("POINT_COLOR"))));
                }
            } else {
                ((ImageView) DetailActivity.circleIndicatorWrap.getChildAt(i)).setImageDrawable((changeDrawableColor(getApplicationContext(), R.drawable.default_dot, MainActivity.clientOptionList.get("POINT_COLOR"))));
            }
        }

        Log.d("scroll", "position  " + position + " // " + scrollX + " // " + pageWidth);
    }
    });



```



</details>


<details>
<summary>EditText</summary>

```java



// EditText MaxLength, MaxLines 강제로 적용해주는 코드 
// 안드로이드 EditText에서 적용해주는 maxLines는 2줄이면 최대 2줄로만 보이고 여러줄을 작성할 수 있음 -> 2줄 넘어가면 스크롤 

    //editText.addTextChangedListener(new MaxLengthAndMaxLinesTextWatcher(14, 2, editText));
    public class MaxLengthAndMaxLinesTextWatcher implements TextWatcher {
        private int maxLength;
        private int maxLines, preIndex;
        private EditText editText;
        private String prevalue;

        public MaxLengthAndMaxLinesTextWatcher(int maxLength, int maxLines, EditText editText) {
            this.maxLength = maxLength;
            this.maxLines = maxLines;
            this.editText = editText;
        }

        @Override
        public void beforeTextChanged(CharSequence s, int start, int count, int after) {
            prevalue = s.toString();
            preIndex = editText.getSelectionStart() - 1;
        }

        @Override
        public void onTextChanged(CharSequence s, int start, int before, int count) {
        }

        @Override
        public void afterTextChanged(Editable s) {
            try {
                if (s.length() > maxLength || editText.getLineCount() > maxLines) {
                    editText.setText(prevalue);
                    editText.setSelection(preIndex);
                }
            } catch (Exception e) {
                Log.e(TAG, "afterTextChanged: " + e.toString());
            }

        }
    }




// EditText 생년월일, 핸드폰 번호, 주민번호 등등 자동으로 - , / 입력해주는 코드 

import android.text.Editable;
import android.text.TextWatcher;
import android.widget.EditText;

public abstract class CustomTextWatcher {

    /* 사용법
     new CustomTextWatcher(EditText, 중간에 들어갈 텍스트 String ,들어가야할 인덱스 new int[]{4, 7}) {
            @Override
            protected void customBeforeTextChanged(CharSequence s, int start, int count, int after) {

            }

            @Override
            protected void customOnTextChanged(CharSequence s, int start, int before, int count) {

            }

            @Override
            protected void customAfterTextChanged(Editable s) {
                ValueCheck(BaseActivity.common.isValidBirth(s.toString().replace(splitString, "")));
            }
        };
    */

    public CustomTextWatcher(EditText editText, String splitString, int[] splitIndex) {
        editText.addTextChangedListener(new TextWatcher() {
            private int _beforeLenght = 0;
            private int _afterLenght = 0;

            @Override
            public void beforeTextChanged(CharSequence s, int start, int count, int after) {
                // 텍스트 변경 전
                // s: 기존 문자열, start: 커서 시작 위치, count: 변경 대상 문자 수, after: 변경 후 문자 수
                _beforeLenght = s.length();
                customBeforeTextChanged(s, start, count, after);
            }

            @Override
            public void onTextChanged(CharSequence s, int start, int before, int count) {
                // 텍스트 변경 시
                // s: 변경된 문자열, start: 커서 시작 위치, before: 변경 대상 문자 수, count: 변경 후 문자 수
                if (s.length() <= 0) {
//                    Log.d("addTextChangedListener", "onTextChanged: Intput text is wrong (Type : Length)");
                    return;
                }

                char inputChar = s.charAt(s.length() - 1);
                if (inputChar != splitString.charAt(0) && (inputChar < '0' || inputChar > '9')) {
                    editText.getText().delete(s.length() - 1, s.length());
//                    Log.d("addTextChangedListener", "onTextChanged: Intput text is wrong (Type : Number)");
                    return;
                }

                _afterLenght = s.length();

                // 삭제 중
                if (_beforeLenght > _afterLenght) {
                    // 삭제 중에 마지막에 splitString 자동으로 지우기
                    if (s.toString().endsWith(splitString)) {
                        editText.setText(s.toString().substring(0, s.length() - 1));
                    }
                }
                // 입력 중
                else if (_beforeLenght < _afterLenght) {
                    for (int i = 0; i < splitIndex.length; i++) {
                        if (_afterLenght == splitIndex[i]) {
                            editText.setText(s.toString().subSequence(0, splitIndex[i]) + splitString + s.toString().substring(splitIndex[i], s.length()));
                        }
                    }
                }
                // 삭제 후 재입력
                for (int i = 0; i < splitIndex.length; i++) {
                    if (start == splitIndex[i] && count == 1) {
                        editText.setText(s.toString().subSequence(0, splitIndex[i]) + splitString + s.toString().substring(splitIndex[i], s.length()));
                    }
                }

                editText.setSelection(editText.length());

                customOnTextChanged(s, start, before, count);
            }

            @Override
            public void afterTextChanged(Editable s) {
                // 텍스트 변경 후
                // s: 변경된 문자열
                customAfterTextChanged(s);
            }
        });
    }
    protected abstract void customBeforeTextChanged(CharSequence s, int start, int count, int after);

    protected abstract void customOnTextChanged(CharSequence s, int start, int before, int count);

    protected abstract void customAfterTextChanged(Editable s);


}





// EditText Enter키 감지 
editTxt.setImeOptions(EditorInfo.IME_ACTION_DONE);
editTxt.setRawInputType(InputType.TYPE_CLASS_TEXT);

editTxt.setOnEditorActionListener(new TextView.OnEditorActionListener() {
            @Override
            public boolean onEditorAction(TextView v, int actionId, KeyEvent event) {
                if (actionId == EditorInfo.IME_ACTION_DONE) {
                    //엔터 입력시 실행 코드
                    txt2_1.performClick();
                }
                return false;
            }
        });
        
// EditText 여러줄 사용, 엔터 막기

    editTxt.setImeOptions(EditorInfo.IME_ACTION_NONE);
    editTxt.setRawInputType(InputType.TYPE_CLASS_TEXT);

    editTxt.setOnKeyListener(new View.OnKeyListener() {
    @Override
    public boolean onKey(View v, int keyCode, KeyEvent event) {
        Log.d(TAG,"enter1 "+keyCode+" // "+event.toString());
        if ((event.getAction() == KeyEvent.ACTION_DOWN) && keyCode == KeyEvent.KEYCODE_ENTER) {
            //엔터 입력시 실행 코드
            Log.d(TAG,"enter1 in22222");
            return true;
        }
        return false;
    }
    });


// EditText 키보드 올라올때 맨 밑에 화면에서 EditText 짤릴 때 

    editPrdDesc.setOnFocusChangeListener(new View.OnFocusChangeListener() {
            @Override
            public void onFocusChange(View view, boolean bool) {
                if(bool){
                    scrollWrap.setPadding(0,0,0,(int)new Common().DpToPx(300,mContext)); //EditText 높이가 300dp
                    scrollWrap.smoothScrollTo(0,scrollWrap.getHeight());
                    Log.d(TAG,bool+" // in"+scrollWrap.getPaddingBottom());
                }else{
                    scrollWrap.setPadding(0,0,0,0);
                    Log.d(TAG,bool+" // in"+scrollWrap.getPaddingBottom());
                }
            }
        });
    // 포커스 들어갈 때 상위 스크롤 뷰에 바텀 패딩주고 포커스 나올 때 없애기.


//EditText 사용중 다른 곳이 눌렸을때 키보드 내리기
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        View focusView = getCurrentFocus();
        if (focusView != null) {
            Rect rect = new Rect();
            focusView.getGlobalVisibleRect(rect);
            int x = (int) ev.getX(), y = (int) ev.getY();
            if (!rect.contains(x, y)) {
                InputMethodManager imm = (InputMethodManager) getSystemService(INPUT_METHOD_SERVICE);
                if (imm != null)
                    imm.hideSoftInputFromWindow(focusView.getWindowToken(), 0);
                focusView.clearFocus();
            }
        }
        return super.dispatchTouchEvent(ev);
    }



```

</details>


<details>
<summary>ImageView </summary>


### 이미지 테스트 url -> https://picsum.photos/200/300?random=인덱스

```java

// ImageView 선택 된 아이템 outline
int tempStroke = (int) common.DpToPx(3, mContext);
if (deleteArray.contains(position)) {
//선택 됨
    GradientDrawable shape = new GradientDrawable();
    shape.setStroke(tempStroke, mContext.getResources().getColor(R.color.point_color));
    ((DataViewHolder) holder).itemView.setForeground(shape);
} else {
//선택 안 됨
    ((DataViewHolder) holder).itemView.setForeground(null);
}

//이미지 색깔 바꿔주는 코드문
  
    public Drawable changeDrawableColor(Context context, int drawable, String newColor) {
    Drawable mDrawable = ContextCompat.getDrawable(context, drawable);
    mDrawable.setColorFilter(new
            PorterDuffColorFilter(Color.parseColor(newColor), PorterDuff.Mode.SRC_IN));
    return mDrawable;
    }

//Glide 
Glide.with(context).load(URL).placeholder(R.mipmap.ic_launcher).error(R.mipmap.ic_launcher).diskCacheStrategy(DiskCacheStrategy.ALL).centerCrop().into(VIEW);
 


```

</details>

<details>
<summary>TextView </summary>

```java


// 글자 색상,사이즈 중복으로 바꾸기

String tempString = "바꾸고 싶은 텍스트";
SpannableString span = SpannableString.valueOf(tempString);
span.setSpan(new RelativeSizeSpan(.9f), 0, 3, Spanned.SPAN_EXCLUSIVE_EXCLUSIVE); // 원래 적용 되있던 코드에서 90% 크기
span.setSpan(new ForegroundColorSpan(activity.getResources().getColor(R.color.point_color)), 4, tempString.length(), Spanned.SPAN_EXCLUSIVE_EXCLUSIVE); // 색상 바꾸기
txtView.setText(span);


// TextView String Shadow 주기
textView.setShadowLayer(1.5f, 0, 0, Color.parseColor("#99000000"));

안드로이드 텍스트 뷰(TextView)는 화면 끝에 위치할 단어가 길어져 더이상 표시할 수 없으면 그 단어를 자동으로 다음 줄로 줄바꿈합니다.<br/>이런 현상을 Word wrap 또는 line wrap라고 불립니다.하지만 자동으로 줄바꿈 되어 보이지 않았으면 하는경우 다음과 같이 코드를 작성합니다.

      myString.replace(" ", "\u00A0");
   
모든 공백을 \u00A0으로 바꾸어 주는데요, u00A0은 No-break space 기호로, 스페이스로 보여지지만 워드 분리를 하지 않기 위한 용도로 사용됩니다.

```

</details>

<details>
<summary>애니메이션</summary>


```java 


안드로이드 애니메이션 사용할 때
    
    반복문에서 사용하면 애니메이션은 무조건 new 생성자로 만들어줄 것..
    하나의 객체를 생성해서 여러 번 사용하려 하면 불규칙적인 에러가 많이 나온다...
    fadeIn =new AlphaAnimation(0, 1);
    fadeOut =new AlphaAnimation(1, 0);
    scaleUp =new ScaleAnimation(0.5f, 1f, 0.5f, 1f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
    scaleDown =new ScaleAnimation(1f, 0.5f, 1f, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);


애니메이션 예시

    ScaleAnimation scaleUp = new ScaleAnimation(1, 1.1f, 1, 1.1f, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);
    ScaleAnimation scaleDown = new ScaleAnimation(1.1f, 1, 1.1f, 1, Animation.RELATIVE_TO_SELF, 0.5f, Animation.RELATIVE_TO_SELF, 0.5f);

    scaleUp.setDuration(300);
    scaleUp.setStartOffset(300);
    ((PlayerViewHolder) holder).itemView.startAnimation(scaleUp);
    scaleDown.setDuration(300);
    scaleUp.setAnimationListener(new Animation.AnimationListener() {
    @Override
    public void onAnimationStart(Animation animation) {

    }

    @Override
    public void onAnimationEnd(Animation animation) {
        ((PlayerViewHolder) holder).itemView.startAnimation(scaleDown);
    }

    @Override
    public void onAnimationRepeat(Animation animation) {

    }
    });

애니메이션 여러개 동시에 사용하기

    AnimationSet set = new AnimationSet(true);
    Animation up = new TranslateAnimation(0, 0, ((PlayerViewHolder) holder).itemView.getHeight(),0);
    up.setDuration(500);
    set.addAnimation(up);
    Animation fade = new AlphaAnimation(0f, 1.0f);
    fade.setDuration(1000);
    set.setStartOffset(150);
    set.addAnimation(fade);

    ((PlayerViewHolder) holder).title.startAnimation(set);
    ((PlayerViewHolder) holder).subTitle.startAnimation(set);
    


ValueAnimator를 사용하여 View 높이 부드럽게 조절

ValueAnimator animator = ValueAnimator.ofInt(View.getHeight(), targetViewHeight);
animator.addUpdateListener(animation -> {
    int animatedValue = (int) animation.getAnimatedValue();
    ConstraintLayout.LayoutParams params = (ConstraintLayout.LayoutParams) View.getLayoutParams();
    params.height = animatedValue;
    MainActivity.binding.webView.setLayoutParams(params);
});
animator.setDuration(0); // 빠른 업데이트를 위해 애니메이션 시간을 0으로 설정
animator.start();
                            


```

</details>

 <details>
<summary>자식 뷰가 부모 뷰보다 크거나 범위를 벗어날 때 기본 값은 짤림 -> 보여주려면 부모 뷰의 clipChildren을 false로 변경 </summary>

```java
//모든 뷰의 setClipChildren를 변경
public void setAllParentsClip(View v) {
        while (v.getParent() != null && v.getParent() instanceof ViewGroup) {
            ViewGroup viewGroup = (ViewGroup) v.getParent();
            viewGroup.setClipChildren(false);
            viewGroup.setClipToPadding(false);
            v = viewGroup;
        }
}
```

</details>


<details>
<summary>유효성 체크 (정규식) </summary>

```java

  public static final String pattern0 = "^(?=.*[A-Z])(?=.*[a-z])(?=.*[0-9])(?=.*[$@$!%*#?&])[A-Za-z[0-9]$@$!%*#?&]{8,32}$"; // 영어 소문자, 영어 대문자, 숫자, 특수문자
    public static final String pattern1 = "^(?=.*[A-Za-z])(?=.*[0-9])(?=.*[$@$!%*#?&])[A-Za-z[0-9]$@$!%*#?&]{8,32}$"; // 영문, 숫자, 특수문자
    public static final String pattern2 = "^[A-Za-z[0-9]]{10,20}$"; // 영문, 숫자
    public static final String pattern3 = "^[[0-9]$@$!%*#?&]{10,20}$"; //영문, 특수문자
    public static final String pattern4 = "^[[A-Za-z]$@$!%*#?&]{10,20}$"; // 특수문자, 숫자
    public static final String pattern5 = "^(19[0-9][0-9]|20\\d{2})(0[0-9]|1[0-2])(0[1-9]|[1-2][0-9]|3[0-1])$"; // 생년월일 8자리
    Matcher match;


    public boolean isValidEmail(String email) {
        boolean err = false;
        String regex = "^[_a-z0-9-]+(.[_a-z0-9-]+)*@(?:\\w+\\.)+\\w+$";
        Pattern p = Pattern.compile(regex);
        Matcher m = p.matcher(email);
        if (m.matches()) {
            err = true;
        }
        return err;
    }

    public boolean isValidBirth(String birth) {
        boolean err = false;
        match = Pattern.compile(pattern5).matcher(birth);
        if (match.find()) {
            err = true;
        }
        return err;
    }

    public boolean pwdRegularExpressionChk(String newPwd) {
        boolean chk = false;
        // 특수문자, 영어 소문자,영어 대문자, 숫자 조합 (8~32 자리)
        match = Pattern.compile(pattern0).matcher(newPwd);
        if (match.find()) {
            chk = true;
        }
        return chk;

       /* // 특수문자, 영문, 숫자 조합 (8~32 자리)
        match = Pattern.compile(pattern1).matcher(newPwd);
        if (match.find()) {
            chk = true;
        }
        return chk;*/

       /* // 영문, 숫자 (10~20 자리)
        match = Pattern.compile(pattern2).matcher(newPwd);
        if(match.find()) {
            chk = true;
        }
        // 영문, 특수문자 (10~20 자리)
        match = Pattern.compile(pattern3).matcher(newPwd);
        if(match.find()) {
            chk = true;
        }
        // 특수문자, 숫자 (10~20 자리)
        match = Pattern.compile(pattern4).matcher(newPwd);
        if(match.find()) {
            chk = true;
        }*/
    }


```

</details>

<details>
<summary>pdf 뷰어 없이 pdf 보기 </summary>
    pdfUrl앞에 -> "http://docs.google.com/gview?embedded=true&url=" 추가 해줄 것.
    "http://docs.google.com/gview?embedded=true&url=" + pdfUrl
</details>



<details>
<summary>StatusBar 투명 만들기 </summary>
   theme.xml에 
    <item name="android:windowTranslucentStatus">true</item>
    <item name="android:windowTranslucentNavigation">true</item>
    추가 후
    
    Activity에
    // In Activity's onCreate() for instance
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            Window w = getWindow();
            w.setFlags(WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS, WindowManager.LayoutParams.FLAG_LAYOUT_NO_LIMITS);
        }
    추가 하기
</details>


<details>
<summary>안드로이드 무선 빌드 </summary>

안드로이드 11이상 버전에서 무선빌드 가능 
    Sdk경로\platform-tools
    C:\Users\user\AppData\Local\Android\Sdk\platform-tools 에서 cmd 키고 
    adb devices 연결된 기기 확인
    adb tcpip 5555 5555포트 개방
    adb connect xxx.xxx.xxx.xxx(ip 주소):5555 //연결
    adb disconnect xxx.xxx.xxx.xxx(ip 주소):5555 //해제
    
    오류 날 때
    adb kill-server
    adb usb
    adb tcpip ... 다시 실행

</details>   
 

<details>
<summary>카카오톡 로그인 API 사용시 디버그와 릴리즈 버전 키 해시 설정 시 유의 사항. </summary>
 *프로가드 사용시*
    proguard 에
    -keep class com.kakao.sdk.**.model.* { <fields>; }
    -keep class * extends com.google.gson.TypeAdapter
    꼭 추가해줄 것.... 이거 안하면 릴리즈 버전에서 로그인 절대 안됨.............................
    

</details>
 


<details>
<summary>12버전 자동으로 스플래쉬 이미지 만들어져서 없애버리는 코드 // 없애는게 아니라 잘 안보이게 하는 코드 </summary>

  themes.xml
    
    <item name="android:windowSplashScreenBackground">@color/white</item>
    <item name="android:windowSplashScreenAnimationDuration">10</item> 
    <item name="android:windowSplashScreenAnimatedIcon">@drawable/drawable_none</item> //drawable_none 빈 drawable 만들어주기
    
</details>
    


<details>
<summary>Glide 원하는 설정 제대로 적용 안될 때 ***** </summary>

  센터크롭 후에 모서리 12  * 순서대로 적용 *
    MultiTransformation multiOption = new MultiTransformation( new CenterCrop(), new RoundedCorners(12) );
    Glide.with(CommunityWriteActivity.this).load(result.getData().getData()).apply(RequestOptions.bitmapTransform(multiOption)).into(imgPicture);
                       
</details>


   
<details>
<summary>플레이스토어 태블릿에서 검색 안될 때 </summary>

    모든 퍼미션들에게 android:required="false" 걸어주기
    <uses-feature android:name="android.hardware.faketouch" android:required="false" />
    <uses-feature android:name="android.hardware.screen.portrait" android:required="false" />
    <uses-feature android:name="android.hardware.telephony" android:required="false" />
    등등 권한 설정에서 false를 걸어줘야 한다. (플레이스토어 출시 개요에서 App Bundle 세부정보보면 확인 가능.)

</details>


  
<details>
<summary>gradle -> dependencies 추가 할 때 옛날 버전 추가하다 보면 에러남 </summary>

      defaultConfig {
            multiDexEnabled true //이거 추가해줄 것
            }
</details>

<details>
<summary>기타</summary>

    프래그먼트 여러개 사용 시 -> replace 대신에 show(), hide() 사용으로 속도 향상 시킬것.
    
    버튼에 그림자 자동으로 들어간거 없애기 xml Button에 style="?android:attr/borderlessButtonStyle" 추가
    
    안드로이드 29 버전 이상 부터는 파일 읽고 쓸때 권한 허용도 하고 이것도 무조건 해줘야 한다... 
    AndroidManifest.xml -> application에 추가 android:requestLegacyExternalStorage="true"  
    
    styles.xml 안에 선언 
    <item name="android:forceDarkAllowed" tools:targetApi="q">false</item> // 다크모드 강제 무시 해주는 코드! 
    setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);//세로모드고정 MainActivity 안에 설정

```java

//클립보드 사용 코드

    ClipboardManager clipboardManager = (ClipboardManager) getSystemService(CLIPBOARD_SERVICE);
    temp = editTxt.getText().toString().trim();
    Log.d(TAG, "temp before : " + temp);
    int ascii = '\n';
    System.out.println("ASCII Numeric Value: " + ascii);
    temp = temp.replaceAll("\n", "\u2800\n"); // 공백을 공간 차지하는 문자로 바꿔주는  코드문 
    
    ClipData clipData = ClipData.newPlainText("CODE", temp); //클립보드에 ID라는 이름표로 id 값을 복사하여 저장
    clipboardManager.setPrimaryClip(clipData);


//상태바 Status바 색깔 바꾸는 코드문

    View view = getWindow().getDecorView();
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    if (view != null) {
        // 23 버전 이상일 때 상태바 하얀 색상에 회색 아이콘 색상을 설정
        view.setSystemUiVisibility(View.SYSTEM_UI_FLAG_LIGHT_STATUS_BAR);
        getWindow().setStatusBarColor(Color.rgb(Integer.parseInt(textColortemp[0]), Integer.parseInt(textColortemp[1]), Integer.parseInt(textColortemp[2])));
    }
    }else if (Build.VERSION.SDK_INT >= 21) {
    // 21 버전 이상일 때
    getWindow().setStatusBarColor(Color.rgb(Integer.parseInt(textColortemp[0]), Integer.parseInt(textColortemp[1]), Integer.parseInt(textColortemp[2])));
    }
                    
// 상태바 없애는 코드문  **setContentView 전에 넣을 것.** 

    getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN,WindowManager.LayoutParams.FLAG_FULLSCREEN);

                    
 //모든 뷰 가져와서 내가 원하는 뷰 있는지 확인하는 코드
 
    ArrayList<View> AllView = getAllChildren(listWrap);

    for(int j=0;j<AllView.size();j++){
        if(AllView.get(j) instanceof TextView){ //모든 뷰에서 텍스트 뷰이면 애니메이션 실행
            Animation animLeftSlide = AnimationUtils.loadAnimation(getApplicationContext(), R.anim.anim_slide_in_right);
            AllView.get(j).startAnimation(animLeftSlide);
        }
    }
 
    private ArrayList<View> getAllChildren(View v) {

    if (!(v instanceof ViewGroup)) {
        ArrayList<View> viewArrayList = new ArrayList<View>();
        viewArrayList.add(v);
        return viewArrayList;
    }

    ArrayList<View> result = new ArrayList<View>();

    ViewGroup vg = (ViewGroup) v;
    for (int i = 0; i < vg.getChildCount(); i++) {

        View child = vg.getChildAt(i);

        ArrayList<View> viewArrayList = new ArrayList<View>();
        viewArrayList.add(v);
        viewArrayList.addAll(getAllChildren(child));

        result.addAll(viewArrayList);
    }
    return result;
    }

 
  
  //btnNext getWidth(), getHeight() 가 0값을 가져올때 뷰가 생성되는걸 기다렸다 가져오는 코드 + 프로그램으로 radius 적용 라운드 처리 

    btnNext.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
        @Override
        public void onGlobalLayout() {
            btnNext.getViewTreeObserver().removeOnGlobalLayoutListener(this);
            GradientDrawable shape = new GradientDrawable();
            Log.d("shape",btnNext.getWidth()+" // "+btnNext.getHeight());
            shape.setCornerRadius(btnNext.getWidth());
            shape.setColor(Color.parseColor(clientOptionList.get("POINT_COLOR")));
            btnNext.setBackground(shape);
            btnNext.setTextColor(Color.parseColor(clientOptionList.get("BG_COLOR")));
        }
    });
    

```

</details>

<details>
<summary>안드로이드 파이어베이스 백그라운드에서 이미지 알림 생성 할 수 있게 해주는 코드 </summary>


### data 로만 받아야함 notification 정보 들어가면 죽음

```java


    try {
        String message = remoteMessage.getData().get("msg");
        String mediaUrl = remoteMessage.getData().get("mediaUrl");

        PendingIntent pendingIntent = PendingIntent.getActivity((Context) this, 0, new Intent(getApplicationContext(), MainActivity.class), PendingIntent.FLAG_UPDATE_CURRENT);
        final String CHANNEL_ID = "LAPPALND_ID";
        NotificationManager mManager = (NotificationManager) getSystemService(Context.NOTIFICATION_SERVICE);
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            final String CHANNEL_NAME = "LAPPALND";
            final String CHANNEL_DESCRIPTION = "LAPPALND CHANNEL";
            final int importance = NotificationManager.IMPORTANCE_HIGH;

            // add in API level 26
            NotificationChannel mChannel = new NotificationChannel(CHANNEL_ID, CHANNEL_NAME, importance);
            mChannel.setDescription(CHANNEL_DESCRIPTION);
            mChannel.enableLights(true);
            mChannel.enableVibration(true);
            mChannel.setVibrationPattern(new long[]{100, 200, 100, 200});
            mChannel.setLockscreenVisibility(Notification.VISIBILITY_PRIVATE);
            mManager.createNotificationChannel(mChannel);
        }

        NotificationCompat.Builder builder = new NotificationCompat.Builder(this, CHANNEL_ID);
        builder.setSmallIcon(R.mipmap.lappland_icon_round);
        builder.setAutoCancel(true);
        builder.setDefaults(Notification.DEFAULT_ALL);
        builder.setWhen(System.currentTimeMillis());
        builder.setSmallIcon(R.mipmap.lappland_icon_round);
        //builder.setContentTitle(title);
        builder.setContentText(message);
        if (!TextUtils.isEmpty(mediaUrl)) {
            Bitmap bit = null;
            try {
                bit = BitmapFactory.decodeStream((InputStream) new URL(mediaUrl).getContent());
            } catch (Exception e) {
            }
            builder.setLargeIcon(bit);

        }
        builder.setContentIntent(pendingIntent);
        builder.setStyle(new NotificationCompat.BigTextStyle().bigText(message));
        //builder.setStyle(new NotificationCompat.BigPictureStyle().bigPicture(bit));
        mManager.notify((int) (System.currentTimeMillis() / 1000), builder.build());
        Log.d("sendNotification", " // " + remoteMessage.getData().toString() + " // " + mediaUrl);

    } catch (Exception e) {

    }

```

</details>

<details>
<summary>안드로이드 키보드 제어 하는 코드문 </summary>


### 키보드 높이 확인 하여 컨트롤 

```java

 public float dpToPx(float value) {
        DisplayMetrics metrics = getResources().getDisplayMetrics();
        return TypedValue.applyDimension(TypedValue.COMPLEX_UNIT_DIP, value, metrics);
    }
    // 전체 뷰 가져와서 키보드 높이 비교하여 키보드 제어 하는 코드  dpToPx 함수!
    final ConstraintLayout fullLayout = findViewById(R.id.fullLayout);
    fullLayout.getViewTreeObserver().addOnGlobalLayoutListener(new ViewTreeObserver.OnGlobalLayoutListener() {
    @Override
    public void onGlobalLayout() {
        int mRootViewHeight = fullLayout.getRootView().getHeight();
        int mConstaritHeight = fullLayout.getHeight();
        int mDiff = mRootViewHeight - mConstaritHeight;
        if (mDiff > dpToPx(200)) {
            btnUpDown.setImageResource(R.drawable.down_btn);
            //Toast.makeText(getApplicationContext(), "키보드 위로", Toast.LENGTH_SHORT).show();
        } else {
            btnUpDown.setImageResource(R.drawable.up_btn);
            //Toast.makeText(getApplicationContext(), "키보드 내려감", Toast.LENGTH_SHORT).show();
        }
    }
    });

```


### EditText 포커스 되어 있을 때 (키보드 올라온 상태)에서 EditText 외에 다른 영역 눌릴 때 키보드 내리기

```java

 // EditText 포커스 사라질 때 키보드 내리기
    @Override
    public boolean dispatchTouchEvent(MotionEvent ev) {
        try {
            View focusView = getCurrentFocus();
            if (focusView != null) {
                Rect rect = new Rect();
                focusView.getGlobalVisibleRect(rect);
                int x = (int) ev.getX(), y = (int) ev.getY();
                if (!rect.contains(x, y)) {
                    InputMethodManager imm = (InputMethodManager) getSystemService(INPUT_METHOD_SERVICE);
                    if (imm != null)
                        imm.hideSoftInputFromWindow(focusView.getWindowToken(), 0);
                    focusView.clearFocus();

                }
            }
        } catch (Exception e) {
            Log.e(TAG, "Excpetion e : " + e.toString());
        }
        return super.dispatchTouchEvent(ev);
    }




```

</details>





   
    
  




    
  
    
    

