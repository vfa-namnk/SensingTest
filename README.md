# 【IoT入門】スマホで加速度と位置情報を取得してクラウドに保存しよう！！
![概要図1](readme-img/top.JPG)
### ★加速度センサー
![概要図2](readme-img/acce.png)
### ★ＧＰＳセンサー
![概要図3](readme-img/gps.png)


## 概要
### マイコンとかセンサとかよく分からなくても楽しめるIoTを目指して・・・
最近ニフクラ mobile backendのハンズオンでIoTを扱うことが増えており、マイコン(Raspberry piやEdison)の使用したハンズオン検討していましたが難易度が高すぎ、初心者の方がついてこれなさそうで困っていました。<br>
「マイコンやGPIOの配線などなど、電子工作と縁遠い人でもIoTを身近に感じ楽しめる内容を・・・」と考えたところ、多くの人が持っているスマホの「センサ」の値をクラウドに保存するコンテンツがよいのではないかと思いつき、今回のコンテンツを作りました。<br>
内容をなるべく簡単にするため「HTML5」と「JavaScript」でスマホアプリの開発ができる「Monaca」を利用し「Andoridユーザー」でも「iPhoneユーザー」でも楽しめる内容としました。

* スマホに元々備わっているセンサーの中でも、加速度センサーとGPSセンサーにアクセスして値を取得し、画面に表示させます。
* そのデータを使って、画面の色を変えてみたり、地図を表示したりします。
* 取得した値をサーバーに保存します。

### ■加速度センサー

![画像1](/readme-img/001.png)

### ■GPSセンサー

![画像2](/readme-img/002.png)

## 事前準備

### 1. [ニフクラ mobile backend](https://mbaas.nifcloud.com/)の会員登録とログイン→アプリ作成

* 上記リンクから会員登録（無料）をします。登録ができたらログインをすると下図のように「アプリの新規作成」画面が出るのでアプリを作成します

![画像3](/readme-img/003.png)

* アプリ作成されると下図のような画面になります
* この２種類のAPIキー（アプリケーションキーとクライアントキー）はXcodeで作成するiOSアプリに[ニフクラ mobile backend](https://mbaas.nifcloud.com/)を紐付けるために使用します

![画像4](/readme-img/004.png)

* この後動作確認でデータが保存される場所も確認しておきましょう

![画像5](/readme-img/005.png)

### 2. Monacaの利用

* [Monaca](https://ja.monaca.io/)にログインします

![画像6](/readme-img/006.png)

* [Monacaデバッガー](https://ja.monaca.io/debugger.html)をインストールします
<br>※今回作成するアプリは動作確認にMonacaデバッガーが必須です！

![画像7](/readme-img/007.png)

* Monacaの新しいプロジェクトを作る

![画像8](/readme-img/008.png)
![画像9](/readme-img/009.png)


* 「インポート方法」の「URLを指定してインポート」をチェックし、下記リンクを右クリックでコピーし、貼り付けます
* プロジェクト：[SensingTest](https://github.com/natsumo/SensingTest/archive/master.zip)

![画像10](/readme-img/010.png)

* ダッシュボードの左側に作成したプロジェクトが追加されています

![画像11](/readme-img/011.png)

* プロジェクトの開発環境が開きます

![画像12](/readme-img/012.png)

### 3. ツールの準備
#### Cordvaプラグインの有効化 [設定済み]
* DeviceMotion 
<br>加速度センサーを利用するためのプラグイン
* Geolocation 
<br>GPSセンサーを利用するためのプラグイン
* Splashscreen 
<br>スプラッシュスクリーンを利用するためのプラグイン


![画像13](/readme-img/013.png)

#### MonacaにJavaScriptSDKを入手する
* ｢JS/CSSコンポーネントの追加と削除…｣を選択します
![画像14](/readme-img/014.png)
* 右上の検索欄に｢NCMB｣と入力して検索ボタンを押す
![画像15](/readme-img/015.png)
* ｢追加｣を選択
![画像16](/readme-img/016.png)
* ｢インストール開始｣を選択（最新のjsバージョンを選択してください）
![画像17](/readme-img/017.png)
* ｢components/ncmb/min.js｣にチェックをつけて｢OK｣を選択
![画像18](/readme-img/018.png)

これで準備完了です！！

## アプリを作る

* コード上の(1)～(11)の番号の書いてある部分は未実装です。
* 必要なコードを書く(コピペ)するとアプリが完成する仕組みになっています！

### 作っていただく部分は…

1. Startボタンを押下時の処理
<br>加速度センサーとGPSセンサーにアクセスして値を取得する
2. Stopボタン押下時の処理
<br>取得したデータをmBaaSに保存する


### 【js/app.js】を編集します

### (1) APIキーの設定

![画像19](/readme-img/019.png)

### (2) mBaaSの初期化
* mBaaSを使うために必ず最初に行います


![画像20](/readme-img/020.png)

```javascript
    ncmb = new NCMB(APPLICATION_KEY,CLIENT_KEY);
```


### 加速度センサーStartボタン押下時の処理
#### (3) 加速度センサーから値を取得する

![画像21](/readme-img/021.png)

```javascript
    if (typeof DeviceMotionEvent.requestPermission === 'function') { // iOS 13+
        DeviceMotionEvent.requestPermission()
            .then(response => {
                if (response == 'granted') {
                    window.addEventListener('devicemotion', onAcceSuccess);
                }
            })
            .catch(console.error);
    } else { // non iOS 13+
        window.addEventListener('devicemotion', onAcceSuccess);
    }
```

### ＧＰＳセンサーStartボタン押下時の処理
#### (4) ＧＰＳセンサーから値を取得する

![画像22](/readme-img/022.png)

```javascript
    navigator.geolocation.getCurrentPosition(onGeoSuccess, onGeoError, geoOption);
```

### Stopボタン押下時の処理 [実装済み]
* 加速度センサー
* ＧＰＳセンサー

![画像23](/readme-img/023.png)

### 加速度の値を保存する【Acce_save_ncmb メソッド】
#### (5) データストアに保存用クラスを作成
* mBaaSのデータストアにデータを保存するためのクラスを作成します

![画像24](/readme-img/024.png)

```javascript
    var AcceData = ncmb.DataStore("AcceData");
```

#### (6) クラスのインスタンスを生成
* 先程作成したクラスのインスタンスを作成します

![画像25](/readme-img/025.png)

```javascript
    var acceData = new AcceData();
```

#### (7) データの保存
* mBaaSのデータストアにデータを保存するためのクラスを作成します

![画像26](/readme-img/026.png)

```javascript
    acceData.set("accelerometer", acce)  
        .save();
```

### GPSの値を保存する 【gps_save_ncmb メソッド】
#### (8) データストアに保存用クラスを作成
#### (9) クラスのインスタンスを生成

![画像27](/readme-img/027.png)

```javascript
    var GpsData = ncmb.DataStore("GpsData");
```

```javascript
    var gpsData = new GpsData();
```

#### (10) 位置情報オブジェクトを作成
#### (11) データの保存

![画像28](/readme-img/028.png)

```javascript
    var geoPoint = new ncmb.GeoPoint(); // (0,0)で生成
    geoPoint.latitude = lat;
    geoPoint.longitude = lng;
```

```javascript
    gpsData.set("geoPoint", geoPoint) 
       .save();
```

### プロジェクトの保存
* ここまでできたら保存をしてください！

![画像29](/readme-img/029.png)

【js/app.js】の編集は完了です！

## 動作確認
### Monacaデバッガーで動かしてみよう！

#### ■加速度センサー
* Startボタンを押すと、x,y,z軸の値を取得し画面に表示します。また、画面に表示された色によって、加速度の変化が簡単に読み取れるようになっています。振ってみてください。
* Stopボタンを押すと、１秒ごとに取得した値（x,y,z軸の値の配列が秒ごとの配列として）がmBaaSに保存されます。

![画像30](/readme-img/030.png)

#### ■GPSセンサー
* Startボタンを押すと、緯度経度の値を取得し画面に表示します。またOpenStreetMapを使用して、地図に表示しています。
* Stopボタンを押すと、値が位置情報（緯度経度の値のペア）としてmBaaSに保存されます。

![画像31](/readme-img/031.png)

### mBaaSの確認

* Monacaデバッガーでの動作確認ができたら、mBaaSのデータストアを見てみましょう！

![画像32](/readme-img/032.png)

クラスに「AcceData」と「GpsData」ができていることを確認

それぞれのクラスを選択！

![画像33](/readme-img/033.png)

mBaaSのクラウド上に保存されました！

### 加速度センサーから値の取得時 [実装済み]

* 成功した場合のコールバック

![画像34](/readme-img/034.png)

```javascript
function onAcceSuccess(event) {
    var acceleration = event.acceleration;
    if (acce_flag) {

        //直前の重力加速度のデータ
        var o_x = 0;
        var o_y = 0;
        var o_z = 0;

        //各軸の加速度データ
        document.acce_js.x.value = acceleration.x;
        document.acce_js.y.value = acceleration.y;
        document.acce_js.z.value = acceleration.z;

        //重力加速度を計算
        //基本的に重力加速度は端末の角度に対し「下」に働く
        o_x = acceleration.x * 0.1 + o_x * (1.0 - 0.1);
        o_y = acceleration.y * 0.1 + o_y * (1.0 - 0.1);
        o_z = acceleration.z * 0.1 + o_z * (1.0 - 0.1);

        //重力加速度を抜いたそれぞれの軸の値
        var last_x = acceleration.x - o_x;
        var last_y = acceleration.y - o_y;
        var last_z = acceleration.z - o_z;

        // センサーの値の変化を色で表示する
        if (Math.abs(last_x) > 20 || Math.abs(last_y) > 20 || Math.abs(last_z) > 20) {
            document.getElementById("color").src = "js/img/red.png";//赤
        } else if (Math.abs(last_x) > 13 || Math.abs(last_y) > 13 || Math.abs(last_z) > 13) {
            document.getElementById("color").src = "js/img/yellow.png";//黄
        } else {
            document.getElementById("color").src = "js/img/blue.png";//青
        }

        var acce = [acceleration.x, acceleration.y, acceleration.z];
        acce_array.push(acce);
    }
};
```

### ＧＰＳセンサーから値の取得時 [実装済み]

* 成功した場合のコールバック

![画像37](/readme-img/037.png)

```javascript
var onGeoSuccess = function (position) {
    if (gps_flag) {
        current = new CurrentPoint();
        //検索範囲の半径を保持する
        current.distance = CurrentPoint.distance;
        //位置情報(座標)を保存する
        current.geopoint = position.coords;
        $(".map").empty();
        writemap(current.geopoint.latitude, current.geopoint.longitude);
        document.gps_js.lat.value = current.geopoint.latitude;
        document.gps_js.lng.value = current.geopoint.longitude;
    }
};
```

* 失敗した場合のコールバック

![画像38](/readme-img/038.png)

```javascript
var onGeoError = function (error) {
    console.log("現在位置を取得できませんでした");
};
```

* 設定するオプション

![画像39](/readme-img/039.png)

```javascript
var geoOption = {
    // 取得する間隔を１秒に設定
    frequency: 1000,
    // 6秒以内に取得できない場合はonGeoErrorコールバックに渡すよう設定
    timeout: 6000
};
```

### 位置情報を保持するクラスを作成

```javascript
function CurrentPoint() {
    // 端末の位置情報を保持する
    geopoint = null;
    // 位置情報検索に利用するための検索距離を指定する
    distance = 0;
}
```

### 位置情報を地図(OpenStreetMap)に表示する

![画像40](/readme-img/040.png)

```javascript
function writemap(lat, lon) {
    // 現在地の地図を表示
    map = new OpenLayers.Map("canvas");
    var mapnik = new OpenLayers.Layer.OSM();
    map.addLayer(mapnik);
    console.log(lat + ":" + lon);
    var lonLat = new OpenLayers.LonLat(lon, lat)
        .transform(
            new OpenLayers.Projection("EPSG:4326"),
            new OpenLayers.Projection("EPSG:900913")
        );
    map.setCenter(lonLat, 15);

    // 現在地にマーカーを立てる
    var markers = new OpenLayers.Layer.Markers("Markers");
    map.addLayer(markers);

    var marker = new OpenLayers.Marker(
        new OpenLayers.LonLat(lon, lat)
            .transform(
                new OpenLayers.Projection("EPSG:4326"),
                new OpenLayers.Projection("EPSG:900913")
            )
    );
    markers.addMarker(marker);
}
```

## まとめ

* Monacaを使って、
<br>■加速度センサー
<br>■ＧＰＳセンサー
<br>に簡単にアクセスできることがわかった！

* mBaaSを使って簡単にデータをクラウドに保存できることがわかった！


## 付録
### *js/app.jsの完成版*
上手く動かなかった場合、以下をコピーして js/app.jsに貼り付けてください。その際、APIキーを書き換えてください。

```javascript
/** アプリケーションキーをかmBaaSからコピーして書き換えてください **/
var APPLICATION_KEY = "YOUR_NCMB_APPLICATION_KEY";
/** クライアントキーをかmBaaSからコピーして書き換えてください **/
var CLIENT_KEY = "YOUR_NCMB_CLIENT_KEY";

var ncmb;
var acce_array;
var acce_flag;
var gps_flag;
var current;

// 起動時に実行
$(function () {
    ncmb = new NCMB(APPLICATION_KEY, CLIENT_KEY);
    acce_array = new Array();
    acce_flag = new Boolean(false);
    gps_flag = new Boolean(false);
    current = null;
});

// 加速度センサーStartボタン押下時の処理
function acce_start() {
    acce_flag = true;
    if (typeof DeviceMotionEvent.requestPermission === 'function') { // iOS 13+
        DeviceMotionEvent.requestPermission()
            .then(response => {
                if (response == 'granted') {
                    window.addEventListener('devicemotion', onAcceSuccess);
                }
            })
            .catch(console.error);
    } else { // non iOS 13+
        window.addEventListener('devicemotion', onAcceSuccess);
    }
}

// 加速度センサーStopボタン押下時の処理
function acce_stop() {
    acce_flag = false;
    window.removeEventListener("devicemotion", onAcceSuccess);
    acce_save_ncmb(acce_array);
    document.acce_js.x.value = null;
    document.acce_js.y.value = null;
    document.acce_js.z.value = null;
    document.getElementById("color").src = "js/img/white.png";
}

// ＧＰＳセンサーStartボタン押下時の処理
function gps_start() {
    gps_flag = true;
    navigator.geolocation.getCurrentPosition(onGeoSuccess, onGeoError, geoOption);
}

// ＧＰＳセンサーStopボタン押下時の処理
function gps_stop() {
    gps_flag = false;
    gps_save_ncmb(current.geopoint.latitude, current.geopoint.longitude);
    document.gps_js.lat.value = null;
    document.gps_js.lng.value = null;
    $(".map").empty();
}

// 加速度の値を保存する
function acce_save_ncmb(acce) {
    var AcceData = ncmb.DataStore("AcceData");
    var acceData = new AcceData();
    acceData.set("accelerometer", acce)
        .save();
}

// GPSの値を保存する
function gps_save_ncmb(lat, lng) {
    var GpsData = ncmb.DataStore("GpsData");
    var gpsData = new GpsData();
    var geoPoint = new ncmb.GeoPoint(); // (0,0)で生成
    geoPoint.latitude = lat;
    geoPoint.longitude = lng;
    gpsData.set("geoPoint", geoPoint)
        .save();
}

// 加速度センサーから値の取得に成功した場合のコールバック
function onAcceSuccess(event) {
    var acceleration = event.acceleration;
    if (acce_flag) {

        //直前の重力加速度のデータ
        var o_x = 0;
        var o_y = 0;
        var o_z = 0;

        //各軸の加速度データ
        document.acce_js.x.value = acceleration.x;
        document.acce_js.y.value = acceleration.y;
        document.acce_js.z.value = acceleration.z;

        //重力加速度を計算
        //基本的に重力加速度は端末の角度に対し「下」に働く
        o_x = acceleration.x * 0.1 + o_x * (1.0 - 0.1);
        o_y = acceleration.y * 0.1 + o_y * (1.0 - 0.1);
        o_z = acceleration.z * 0.1 + o_z * (1.0 - 0.1);

        //重力加速度を抜いたそれぞれの軸の値
        var last_x = acceleration.x - o_x;
        var last_y = acceleration.y - o_y;
        var last_z = acceleration.z - o_z;

        // センサーの値の変化を色で表示する
        if (Math.abs(last_x) > 20 || Math.abs(last_y) > 20 || Math.abs(last_z) > 20) {
            document.getElementById("color").src = "js/img/red.png";//赤
        } else if (Math.abs(last_x) > 13 || Math.abs(last_y) > 13 || Math.abs(last_z) > 13) {
            document.getElementById("color").src = "js/img/yellow.png";//黄
        } else {
            document.getElementById("color").src = "js/img/blue.png";//青
        }

        var acce = [acceleration.x, acceleration.y, acceleration.z];
        acce_array.push(acce);
    }
};

// ＧＰＳセンサーから位置情報の取得に成功した場合のコールバック
var onGeoSuccess = function (position) {
    if (gps_flag) {
        current = new CurrentPoint();
        //検索範囲の半径を保持する
        current.distance = CurrentPoint.distance;
        //位置情報(座標)を保存する
        current.geopoint = position.coords;
        $(".map").empty();
        writemap(current.geopoint.latitude, current.geopoint.longitude);
        document.gps_js.lat.value = current.geopoint.latitude;
        document.gps_js.lng.value = current.geopoint.longitude;
    }
};

// ＧＰセンサーから位置情報の取得に失敗した場合のコールバック
var onGeoError = function (error) {
    console.log("現在位置を取得できませんでした");
};

// ＧＰＳセンサーから位置情報をする時に設定するオプション
var geoOption = {
    // 取得する間隔を１秒に設定
    frequency: 1000,
    // 6秒以内に取得できない場合はonGeoErrorコールバックに渡すよう設定
    timeout: 6000
};

// 位置情報を保持するクラスを作成
function CurrentPoint() {
    // 端末の位置情報を保持する
    geopoint = null;
    // 位置情報検索に利用するための検索距離を指定する
    distance = 0;
}

// 位置情報を地図(OpenStreetMap)に表示する
function writemap(lat, lon) {
    // 現在地の地図を表示
    map = new OpenLayers.Map("canvas");
    var mapnik = new OpenLayers.Layer.OSM();
    map.addLayer(mapnik);
    console.log(lat + ":" + lon);
    var lonLat = new OpenLayers.LonLat(lon, lat)
        .transform(
            new OpenLayers.Projection("EPSG:4326"),
            new OpenLayers.Projection("EPSG:900913")
        );
    map.setCenter(lonLat, 15);

    // 現在地にマーカーを立てる
    var markers = new OpenLayers.Layer.Markers("Markers");
    map.addLayer(markers);

    var marker = new OpenLayers.Marker(
        new OpenLayers.LonLat(lon, lat)
            .transform(
                new OpenLayers.Projection("EPSG:4326"),
                new OpenLayers.Projection("EPSG:900913")
            )
    );
    markers.addMarker(marker);
}
```
