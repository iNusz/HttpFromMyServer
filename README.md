# HttpFromMyServer


내가 만든 서버에 있는 데이터를 안드로이드에 출력해보자.



<br/>



## Permission


인터넷을 사용하기 때문에 Permission을 Manifest에 설정해준다.



```xml
<uses-permission android:name="android.permission.INTERNET"/>
```


<br/>



## MainActivity


메인에서는 networkTask를 호출해준다.


```java
// 내 서버의 데이터를 요청

networkTask("http://127.0.0.1:8080/bbb.jsp");    

이때 서버주소는 내 ip주소를 적는다.
```



<br/>




#### getData



서버의 요청처리와 응답처리를 나타내준다.


파일I/O에서 속도를 향상시키기 위해 Buffer가 쓰였고, readLine으로 읽었다.


StringBuilder을 써서 문자열의 계산이 더 빨라지도록 만들었다.



```java
public void getData(String url) {
  StringBuilder result = new StringBuilder();
  try {
    // 1. 요청처리
    URL serverUrl = new URL(url);
    // 주소에 해당하는 서버의 socket을 연결
    HttpURLConnection con = (HttpURLConnection)serverUrl.openConnection();
    // OutputStream으로 데이터를 요청
    // Http 통신 방법 중에 GET으로 하겠다는걸 알려준다
    con.setRequestMethod("GET");

    // 2. 응답처리

    // 응답의 유효성 검사 (파일이 없으면 400번대 404, 에러가 나면 500번대, 성공하면 200)
    int responseCode = con.getResponseCode();
    if (responseCode == 200) {   //200 대신 HttpURLConnection.HTTP_OK 로 해도 된다

      BufferedReader br = //줄단위로 데이터를 읽기 위해서 버퍼에 담는다
      new BufferedReader(new InputStreamReader(con.getInputStream()));

      // 버퍼로 된거 읽기
      String temp = "";         // 결과값을 담아서 처리한다. 왜냐하면 readLine만 해버리면 읽은다음에 사용할 방법이 없기 때문
      while((temp = br.readLine()) != null){
        result.append(temp+"\n"); // 문자열의 합을 StringBuilder에서는 append라는걸 지원 해 준다. temp라는 글자가 쌓인다
        // 또 readLine을 하는순간 줄바꿈이 다날라가서 인위적으로 줄을 만들어줘야한다.
      }

    }
  } catch (Exception e) {
    e.printStackTrace();
  }
}
```


<br/>



#### networkTask


서버는 서브쓰레드에서 구동해줘야 하기 때문에 Thread 중에 AsyncTask를 사용한다.


```java
public void networkTask(String url){
  new AsyncTask<String, Void, String>(){
    // 데이터를 가져오고 아래에 result값을 받아온다
    @Override
    protected String doInBackground(String... params) {
      String result = getData(params[0]);
      return result; // result 를 onPostExecute로 넘겨주기 위해 void타입을 바꿔야한다
    }
    // 화면에 뿌려준다
    @Override
    protected void onPostExecute(String r) { //void에서 위에서 받아와야하기 때문에 string으로 바꾼다
      // super.onPostExecute(r);
      Log.e("Result","결과 :"+r);
    }
  }.execute(url);
}
```
