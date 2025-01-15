# Retrofit 🛜
Je knjižnica, ki poenostvi delo z HTTP zahtevki med našo android aplikacijo in http strežnikom. Grajen je nad knjižico okHttp.

# Prednosti in slabosti 
- ✅ V večini primerov manj kode
- ✅ Hitrejše doddajanje novih klicev
- ✅ Podpira JSON de/serializacijo (s pomočjo Gson, Moshi, Kotlin Serialization, ...)
- ✅ Enostavna podpora sinhronih in asinhronih klicev
- 👎 Večja velikost knjižice, kot okHttp
- 👎 Ne podpira spletnih vtičev (WebSocket)
- 👎 Če nisi seznanjen z knjižico, večja kompleksnost kode

# Licenca ⚖️
**Apache Licenca 2:**
- Prosta uporaba: Programsko opremo lahko uporabljate za kateri koli namen.
- Spreminjanje: Programsko opremo lahko spreminjate in ustvarjate izpeljana dela.
- Distribucija: Originalne ali spremenjene različice programske opreme lahko distribuirate.
- Komercialna uporaba: Programsko opremo lahko uporabljate v komercialne namene.
- Brez plačil: Za nobeno od teh uporab ni potrebno plačevati licenčnin ali drugih pristojbin.

# Popularnost 📈
Njihov GitHub repozitorij ima 43.2K zvezdic in 7.3K forkov in je tako ena izmed najpopularnejši knjižic v androidu.

# Vzrdževanje projekta 🧑‍🏫
Projekt ima 160 razvijalcev, zadnja sprememba (commit), pa je bila pred 4 dnevi.

# Kako ga doddati v projekt z Gradle? 🤔
Odpremo gradle datoteko naše aplikacije in doddamo:
```
dependencies {
    implementation 'com.squareup.retrofit2:retrofit:2.9.0'
    implementation 'com.squareup.retrofit2:converter-gson:2.9.0' // Če želimo JSON de/serializacijo z Gson
}
```

# Kako deluje? 🤔
1. Naredimo Retrofit instanco
```
object RetrofitInstance {
    private const val BASE_URL = "http://11.11.11.111:3000" // Replace with your server URL

    // included for debugging
    private val loggingInterceptor = HttpLoggingInterceptor().apply {
        level = HttpLoggingInterceptor.Level.BODY // Options: NONE, BASIC, HEADERS, BODY
    }

    private val httpClient = OkHttpClient.Builder()
        .addInterceptor(loggingInterceptor)
        .build()

    val api: ApiService by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .client(httpClient)
            .addConverterFactory(GsonConverterFactory.create(Gson()))
            .build()
            .create(ApiService::class.java)
    }
}
```
2. Defeniramo zahtevke, ki jih želimo poslati na strežnik
Zahtveke v osnovi defeniramo tako, da se v anotaciji funkcije odločimo za tip zahtevka (GET, POST, DELETE, PUT, ...) ter povemo pot kam želimo zahtevek poslati
```
@GET("/path") // v tem primeru bi celotna pot zahtevka izgledala npr. tako: "http://11.11.11.111:3000/path" 
```
sledi funkcija povezana z dano anotacijo, ki jo bomo kasneje lahko klicali v kodi, npr.
```
fun getMessageFromServer(): Call<ServerResponse>
```
pri čemer moramo z razredom ServerResponse, povedati kakšen odgovor pričakujemo od strežnika, npr.
```
data class ServerResponse(
    val message: String
)
```
če želimo v naslov zahteveka doddati parameter to storimo tako:
```
@GET("/path/{id}")
fun getMessageFromServer(@Path("id") id: String): Call<ServerResponse>
```
v primeru uporabe POST-a, želimo zahtevku doddati telo, za to bomo potrebovali doddatni razred MyMessage, ki bo dal našemu telesu obliko:
```
data class MyMessage(
    val message: String
)
```
zahtevek bi potem lahko izgledal tako:
```
@POST("/path")
fun postMessageToServer(@Body msg : MyMessage) : Call<ServerResponse>
```
Če želimo spremeniti glavo zahtevka lahko to storimo npr. tako:
```
@GET("/path")
fun getMessageFromServer(@Header("Authorization") token: String): Call<ServerResponse>
```
tako pa bi lahko izgledal primer ko želimo na strežnik poslati sliko:
```
@Multipart
@POST("/path")
fun postImage(@Part image: MultipartBody.Part) : Call<ServerResponse>
```
Vse naše zahtevke zapišemo v nek vmesnik:
```
interface ApiService {
    @GET("/path")
    fun getMessageFromServer(): Call<ServerResponse>

    @GET("/path/{id}")
    fun getMessageFromServer(@Path("id") id: String): Call<ServerResponse>

    @GET("/path")
    fun getMessageFromServer(@Header("Authorization") token: String): Call<ServerResponse>

    @Multipart
    @POST("/path")
    fun postImage(@Part image: MultipartBody.Part) : Call<ServerResponse>
}
```
3. Implementirajmo dane funkcije, primer za getMessageFromServer():
```
RetrofitInstance.api.getMessageFromServer().enqueue(object : Callback<ServerResponse> {
    override fun onResponse(call: Call<ServerResponse>, response: Response<ServerResponse>) {
        if (response.isSuccessful) {
            // Handle the successful response
            val serverResponse = response.body()
            Log.d("API", "Message from server: ${serverResponse?.message}")
        } else {
            // Handle error response
            Log.e("API", "Error: ${response.errorBody()?.string()}")
        }
    }

    override fun onFailure(call: Call<ServerResponse>, t: Throwable) {
        // Handle failure (e.g., network issues)
        Log.e("API", "Request failed: ${t.message}")
    }
})
```






