---
title: Ruby On Rails] Daum 지도 API 활용하기
tags:
- Ruby
- Daum지도
categories:
- Ruby On Rails
---
다음 지도 API를 이용해 키워드를 검색하여 위치를 표시하고 선택하여 D/B에 위도와 경도를 저장해 선택한 위치를 불러온다.
<!-- more -->

## 다음 지도 API Key 불러오기
- API Key 받기\\
다음 지도를 이용하려면 우선 [카카오개발자](https://developers.kakao.com/)사이트에 로그인하여 API Key를 지급받아야한다.
  ![Image Alt 텍스트](/assets/images/RubyOnRails/daum_map_01.png){:class="img-responsive"}<br>
-- 로그인을 한 후 내 애플리케이션으로 이동한다
![Image Alt 텍스트](/assets/images/RubyOnRails/daum_map_02.png){:class="img-responsive"}
-- 내 애플리케이션으로 이동후 앱 만들기를 클릭해 다음과 같이 자신의 애플리케이션을 생선한다
![Image Alt 텍스트](/assets/images/RubyOnRails/daum_map_03.png){:class="img-responsive"}
-- 플랫폼을 웹으로 설정한 후 개발 환경의 도메인을 입력후 저장한다.\\
-- JavaScript키를 메모해둔다

## 지도 생성하기
- 키워드 검색이 가능한 다음 지도 불러오기\\
[다음 지도 API](http://apis.map.daum.net/web/sample/keywordList/)
- 검색 결과를 클릭하여 좌표 받아오기
~~~ javascript
(function(marker, title) {
            daum.maps.event.addListener(marker, 'mouseover', function() {
                displayInfowindow(marker, title);
            });

            daum.maps.event.addListener(marker, 'mouseout', function() {
                infowindow.close();
            });

            itemEl.onmouseover =  function () {
                displayInfowindow(marker, title);
            };

            itemEl.onmouseout =  function () {
                infowindow.close();
            };
        })(marker, places[i].place_name);
~~~
이 부분을
~~~ javascript
(function(marker, title) {
          daum.maps.event.addListener(marker, 'click', function(mouseEvent) {
              displayInfowindow(marker, title);
              var latlng = marker.getPosition();

              var message = '클릭한 위치의 위도는 ' + latlng.getLat() + ' 이고, ';
              message += '경도는 ' + latlng.getLng() + ' 입니다';

              var resultDiv = document.getElementById('clickLatlng');
              resultDiv.innerHTML = message;

              var resultLat = document.getElementById('Lat');
              var resultLng = document.getElementById('Lng');
              resultLat.value = latlng.getLat();
              resultLng.value = latlng.getLng();
          });

          // daum.maps.event.addListener(marker, 'mouseout', function() {
          //     infowindow.close();
          // });

          itemEl.onclick =  function () {
              displayInfowindow(marker, title);
              var latlng = marker.getPosition();

              var message = '클릭한 위치의 위도는 ' + latlng.getLat() + ' 이고, ';
              message += '경도는 ' + latlng.getLng() + ' 입니다';

              var resultDiv = document.getElementById('clickLatlng');
              resultDiv.innerHTML = message;

              var resultLat = document.getElementById('Lat');
              var resultLng = document.getElementById('Lng');
              resultLat.value = latlng.getLat();
              resultLng.value = latlng.getLng();
          };

          // itemEl.onmouseout =  function () {
          //     infowindow.close();
          // };
      })(marker, places[i].place_name);
~~~
이렇게 수정한다.
또한 HTML body부분도 수정해주어야한다.

~~~ html
<div id="clickLatlng"></div>
<input type="text" id="Lat">
<input type="text" id="Lng">
~~~
이 부분을 추가하여 결과 값들이 보일수 있도록 한다.
![Image Alt 텍스트](/assets/images/RubyOnRails/daum_map_04.png){:class="img-responsive"}
이와 같이 마커나 목록을 클릭했을시 해당 위치의 좌표 값을 받아올수 있다.
이제 이 값을 hidden태그로 D/B에 저장하여 필요할때 좌표를 불러와 해당 지역을 표시할 수 있다.
## 지도 불러오기

~~~ ruby
# maps#show
def show
  @map = Map.find params[:id]
end
~~~

Show 컨트롤러를 작성하고 Show.html.erb를 생성한다.

~~~ html
show.html.erb

<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8">
    <title>마커 생성하기</title>
</head>
<body>
<div id="map" style="width:100%;height:350px;"></div>

<script type="text/javascript" src="//dapi.kakao.com/v2/maps/sdk.js?appkey= !!!! 자신의 API Key를 넣어주세요 !!!!"></script>
<script>
var mapContainer = document.getElementById('map'), // 지도를 표시할 div
    mapOption = {
        center: new daum.maps.LatLng(<%= @map.lat %>, <%= @map.lng %>), // 지도의 중심좌표
        level: 3 // 지도의 확대 레벨
    };

var map = new daum.maps.Map(mapContainer, mapOption); // 지도를 생성합니다

// 마커가 표시될 위치입니다
var markerPosition  = new daum.maps.LatLng(<%= @map.lat %>, <%= @map.lng %>);

// 마커를 생성합니다
var marker = new daum.maps.Marker({
    position: markerPosition
});

// 마커가 지도 위에 표시되도록 설정합니다
marker.setMap(map);

// 아래 코드는 지도 위의 마커를 제거하는 코드입니다
// marker.setMap(null);
</script>
</body>
</html>
~~~

여기서도 마찬가지로 자신의 API Key를 입력후 인자로 D/B에 저장돼어있는 lat값과 lng값을 넘겨줘서 지정한 위치의 마커를 생성한다.

![Image Alt 텍스트](/assets/images/RubyOnRails/daum_map_05.png){:class="img-responsive"}
