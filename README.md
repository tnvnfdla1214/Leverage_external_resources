# 외부 리소스 활용(API 활용)
안드로이드를 개발하면서 날씨나 교통정보등 공공 데이터를 사용할 때가 있습니다.사용하는 방법에 대해서 기록해보겠습니다. 아래의 설명은 코로나 센터 서비스에 대한 기록입니다.
### :wrench: 프로젝트 사용 사례
<p align = center>
<img src = "https://user-images.githubusercontent.com/48902047/163483997-9a07ed56-6348-497c-9521-46c98d3e038f.jpg" width="20%" height="20%">
<img src = "https://user-images.githubusercontent.com/48902047/163483987-b23ea8c2-f069-4109-a193-847e5a53c9b1.jpg" width="20%" height="20%">
<img src = "https://user-images.githubusercontent.com/48902047/163483969-226570af-e232-4f1a-a35d-6ec0ec8986a9.jpg" width="20%" height="20%">
<img src = "https://user-images.githubusercontent.com/48902047/163483977-382c6b2e-0fcf-4a84-b4ca-68ab50224596.jpg" width="20%" height="20%">
</p>

+ [코로나 센서 서비스](https://github.com/tnvnfdla1214/Covid19_Map)
  + [공공데이터_코로나19 예방접종 센터_API](https://www.data.go.kr/tcs/dss/selectApiDataDetailView.do?publicDataPk=15077586)
  + 네이버 지도
+ [배달의 민족 클론 코딩](https://github.com/tnvnfdla1214/DelevryProject)
  + [SK_위치기반 주변 음식점 정보_API](https://openapi.sk.com/content/API)
  + 구글 지도
+ [Arbnb 프로젝트 클론 코딩](https://github.com/tnvnfdla1214/Airbnb_project)
  + 네이버 지도
+ [깃허브 레파지토리 클론 코딩](https://github.com/tnvnfdla1214/github_repository)

***
### :lollipop: 설명

#### API 신청하기
![image](https://user-images.githubusercontent.com/48902047/169691361-3c61f936-cacb-47be-9313-d7a7ce11db8c.png)
![image](https://user-images.githubusercontent.com/48902047/169691891-4b659fc5-bc98-4c67-918e-958791b9eb8b.png)


#### gradle_properties 에 Key_ID 추가
![image](https://user-images.githubusercontent.com/48902047/169691736-53b021a6-8bf8-4bf6-9ce2-698ec131de37.png)
이렇게 저장하는 이유는 key번호를 직접 꺼내지 않으므로서 관리하기도 쉽고 비밀로 하기 위해서입니다.
#### gradle_gradle(Module)에 API_KEY,API_SECRET 설정
![image](https://user-images.githubusercontent.com/48902047/169691816-632332a4-8327-4f52-94ea-ef519e8a2f28.png)
#### Manifsst 에 meta-data 추가
![image](https://user-images.githubusercontent.com/48902047/169691863-9b390c5c-5357-432c-975b-6028bbc36e3a.png)
#### url 오브젝트 설정
```Kotlin
/*url.kr*/
object Url {
    const val BASE_URL = "https://api.odcloud.kr/api/15077586/v1/centers/"
}
```
#### API 데이터를 data 클래스로 Json -> kotlin 변경
![image](https://user-images.githubusercontent.com/48902047/169692011-b28e6f05-8e53-4a46-95a5-9d64758dc603.png)
![image](https://user-images.githubusercontent.com/48902047/169692023-f1b4ce7f-9bd8-4051-81ac-1da1a19199fa.png)
#### Hilt(DI)로 종속성 설정
```Kotlin
/*APIModule.kr*/
@Module
@InstallIn(SingletonComponent::class)
object ApiModule {
    @Provides
    fun provideBaseUrl() = Url.BASE_URL
    @Singleton
    @Provides
    fun provideOkHttpClient() = if (BuildConfig.DEBUG) {
        val loggingInterceptor = HttpLoggingInterceptor()
        loggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY)
        OkHttpClient.Builder()
            .addInterceptor(loggingInterceptor)
            .build()
    } else {
        OkHttpClient.Builder().build()
    }
    @Singleton
    @Provides
    fun provideRetrofit(okHttpClient: OkHttpClient): Retrofit {
        return Retrofit.Builder()
            .client(okHttpClient)
            .baseUrl(provideBaseUrl())
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }
    @Singleton
    @Provides
    fun provideApiService(retrofit: Retrofit): RetrofitInterface {
        return retrofit.create(RetrofitInterface::class.java)
    }
}
```
#### API에서 정보 받아오기
아래 예시는 Repository에서 받아오는 코드입니다.
```Kotlin
/*ServiceRepository.kr*/
class ServiceRepositoryImpl (
    private val covidDao: CovidDao,
    private val api: RetrofitInterface,
    private val ioDispatcher: CoroutineDispatcher,
    private val mainDispatcher: CoroutineDispatcher
) : ServiceRepository {

    override suspend fun getCenterList(count: MutableLiveData<Int>) {
        val responseData: MutableLiveData<CentersApi> = MutableLiveData()
        for(i in 1..10){
            api.getCovidCenter(i + 1, 10).enqueue(object : Callback<CentersApi> {
                override fun onResponse(
                    call: Call<CentersApi>,
                    response: Response<CentersApi>
                ) {
                    if (response.isSuccessful) {
                        // code == 200
                        responseData.value = response.body()//결과값 받아오기
                        .
                        .
                }
                override fun onFailure(call: Call<CentersApi>, t: Throwable) {
                    //실패
                    Log.d("getCenterList", t.toString())
                    .
                    .
                }
            })
        }
    }
}
```
