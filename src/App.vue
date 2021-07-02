<template>
  <div class="container">
    <div
      class="banner dark-translucent-bg"
      style="
        background-image: url('https://www.halton.com/wp-content/uploads/2020/05/Concert_hall_rock_concert-1920x938.jpg');
        background-position: 50% 50%;
      "
    >
      <!-- breadcrumb end  -->
      <div class="container" style="height: 50">
        <div class="row justify-content-lg-center">
          <div class="col-lg-8 text-center pv-20">
            <h2 class="title">
              <strong>문 화 지 도2 </strong>
            </h2>
            <div class="separator"></div>
            <p class="text-center" data-effect-delay="100">Moonhwa Joa</p>
          </div>
        </div>
      </div>
    </div>
    <div class="row mt-2">
      <div class="col">
        <div class="header-top dark rounded">
          <button class="btn-secondary m-1" @click="myLocation">내 위치</button>
          <button class="btn-secondary m-1" @click="nearLoc">주변검색</button>
          <select class="btn-dark rounded" name="gu" v-model="gu" @change="showMapGu">
            <option selected value="">[ 구 선택 ]</option>
            <option value="강남">강남</option>
            <option value="강북">강북</option>
            <option value="강동">강동</option>
            <option value="강서">강서</option>
            <option value="관악">관악</option>
            <option value="광진">광진</option>
            <option value="구로">구로</option>
            <option value="금천">금천</option>
            <option value="노원">노원</option>
            <option value="도봉">도봉</option>
            <option value="동대문">동대문</option>
            <option value="동작">동작</option>
            <option value="마포">마포</option>
            <option value="서대문">서대문</option>
            <option value="서초">서초</option>
            <option value="성동">성동</option>
            <option value="성북">성북</option>
            <option value="송파">송파</option>
            <option value="양천">양천</option>
            <option value="영등포">영등포</option>
            <option value="용산">용산</option>
            <option value="은평">은평</option>
            <option value="종로">종로</option>
            <option value="중구">중구</option>
            <option value="중랑">중랑</option>
          </select>
          <button
            id="sbtn"
            class="btn-secondary mt-1 mr-2"
            style="float: right"
            @click="searchName"
          >
            검색
          </button>
          <input
            id="sinput"
            class="btn-dark mt-1"
            style="float: right"
            placeholder="검색어 입력"
            v-model="search"
            @keyup.enter="searchName"
          />

          <div id="map" class="map"></div>
          <div class="mt-3 form-check-inline ml-2">
            <input type="checkbox" id="check1" v-model="checks[1]" @change="renderChecked" />
            <label for="check1">공연장</label>
          </div>
          <div class="mt-3 form-check-inline ml-2">
            <input type="checkbox" id="check3" v-model="checks[3]" @change="renderChecked" />
            <label for="check3">박물관/기념관</label>
          </div>
          <div class="mt-3 form-check-inline ml-2">
            <input type="checkbox" id="check4" v-model="checks[4]" @change="renderChecked" />
            <label for="check4">미술관</label>
          </div>
          <div class="mt-3 form-check-inline ml-2">
            <input type="checkbox" id="check6" v-model="checks[6]" @change="renderChecked" />
            <label for="check6">예술회관</label>
          </div>
          <div class="mt-3 form-check-inline ml-2">
            <input type="checkbox" id="check7" v-model="checks[7]" @change="renderChecked" />
            <label for="check7">문화원</label>
          </div>
          <div class="mt-3 form-check-inline ml-2">
            <input type="checkbox" id="check8" v-model="checks[8]" @change="renderChecked" />
            <label for="check8">도서관</label>
          </div>
          <div class="mt-3 form-check-inline ml-2">
            <input type="checkbox" id="check11" v-model="checks[11]" @change="renderChecked" />
            <label for="check11">기타</label>
          </div>
        </div>
      </div>
      <div class="col">
        <div v-if="datas.length == 0">
          <hr class="p-0 m-0" />
          <a
            class="nav-link btn-outline-secondary"
            onclick="this.nextSibling.style.display=(this.nextSibling.style.display=='none')?'block':'none';"
            href="javascript:void(0)"
            style="color: black"
            >검색 결과가 없습니다.
          </a>
          <hr class="p-0 m-0" />
        </div>
        <div v-for="(data, index) in datas" :key="index" @click="movemapto(data)" class="m-1">
          <hr class="p-0 m-0" />
          <a
            class="nav-link btn-outline-secondary"
            onclick="this.nextSibling.style.display=(this.nextSibling.style.display=='none')?'block':'none';"
            href="javascript:void(0)"
            style="color: black"
          >
            [{{ data.codename }}] {{ data.fac_name }}
          </a>
          <div class="m-3" style="display: none">
            <ul class="list-group">
              <li class="list-group-item">주소: {{ data.addr }}</li>
              <li class="list-group-item" v-if="data.phne !== null">번호: {{ data.phne }}</li>
              <li class="list-group-item" v-if="data.homepage !== null">
                홈페이지:{{ data.homepage }}
              </li>
              <li class="list-group-item" v-if="data.closeday !== null">
                휴무일:{{ data.closeday }}
              </li>
              <li class="list-group-item" v-if="data.etc_desc !== null">
                상세정보:{{ data.etc_desc }}
              </li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
var markers = []; //마커정보를담을 배열
let filteredData = [];
let datas = []; //데이터(문화시설정보)를 담을 배열
var map; //카카오 MAP API 객체를 저장할 변수
var my_x, my_y;
var search = '';
var checks = [];
function addMarker(data) {
  if (checks[data.subjcode]) {
    //data를 받아온다.
    // 마커를 생성합니다
    var imageSize = new kakao.maps.Size(40, 40); // 마커이미지의 크기입니다
    var imageOption = {}; // 마커이미지의 옵션입니다. 마커의 좌표와 일치시킬 이미지 안에서의 좌표를 설정합니다.
    var imageSrc =
      'https://cdn2.iconfinder.com/data/icons/metaphoricon-essentials-vol-1/128/building-government-bank-municipal-public-hall-library-128.png';
    if (data.codename.includes('공연')) {
      imageSrc =
        'https://cdn1.iconfinder.com/data/icons/rock-and-roll-11/64/stage-performance-show-live-concert-128.png'; // 마커이미지의 주소입니다
    } else if (data.codename.includes('박물관')) {
      imageSrc =
        'https://cdn1.iconfinder.com/data/icons/university-indigo-vol-2/256/Museum-256.png';
    } else if (data.codename.includes('미술')) {
      imageSrc =
        'https://cdn0.iconfinder.com/data/icons/lifestyle-entertainment-vol-2/512/museum_art_painting_artist-256.png';
    } else if (data.codename.includes('도서')) {
      imageSrc =
        'https://cdn3.iconfinder.com/data/icons/office-supply-essentials-colored/48/JD-46-128.png';
    } else if (data.codename.includes('문화')) {
      imageSrc =
        'https://cdn2.iconfinder.com/data/icons/landmark3-5/64/Romanian-athenaeum-concert-hall-tourism-256.png';
    } else if (data.fac_name.includes('체육' || '스포츠 ')) {
      imageSrc =
        'https://cdn3.iconfinder.com/data/icons/sports-210/100/gymnasium-fitness-sports-gym-club-fitness-center-weightlifting-club-gymnasium-building-256.png';
    }
    // https://cdn3.iconfinder.com/data/icons/human-resources-2-6/48/163-256.png

    // 마커의 이미지정보를 가지고 있는 마커이미지를 생성합니다
    var markerImage = new kakao.maps.MarkerImage(imageSrc, imageSize, imageOption);

    var marker = new kakao.maps.Marker({
      //data의 좌표를 이용해 마커를 생성
      position: new kakao.maps.LatLng(data.x_coord, data.y_coord),
      //클릭 가능하도록 설정
      image: markerImage, // 마커이미지 설정
      clickable: true,
    });

    // 마커가 지도 위에 표시되도록 설정합니다
    marker.setMap(map);

    //마커의 윈도우컨텐트를 만든다. 받아온 data의 정보를 뿌린다.
    var iwContent = `
  <div style="padding:2px; color:black;"> ${data.fac_name} </div>
  <div style="padding:2px; color:black;"> ${data.phne} </div>
  <a style="padding:2px; color:black;" href= "${data.homepage}" target='_blank'>${data.homepage}</a><br/>
  <div style="padding:2px; color:black;"> ${data.addr} </div>

  `,
      iwRemoveable = true; // removeable 속성을 ture 로 설정하면 인포윈도우를 닫을 수 있는 x버튼이 표시됩니다
    //인포윈도우를  만든다.
    var infowindow = new kakao.maps.InfoWindow({
      content: iwContent,
      removable: iwRemoveable,
    });
    //마커의 클릭 이벤트를 설정한다.
    kakao.maps.event.addListener(marker, 'click', function () {
      // 마커 위에 인포윈도우를 표시합니다
      infowindow.open(map, marker);
    });
    // 생성된 마커를 배열에 추가합니다
    markers.push(marker);
    // 생성된 마커의 데이터를 data배열에 저장한다(리스트에서 꺼내보기위함.)
    datas.push(data);
  }
}
//json파일 임포트
import cultures from './resources/data/culture.json';
export default {
  data() {
    return {
      lat: 37.49682, // X 좌표
      lng: 127.05234, // Y좌표
      gu: '', //구(default = '')
      datas: [], //v-for를 이용해 리스트를 뽑아올 배열
      cultures: cultures.datas, //Json파일에서 뽑아온 데이터
      search: '',
      checks: [true, true, true, true, true, true, true, true, true, true, true, true],
    };
  },
  mounted() {
    if (window.kakao && window.kakao.maps) {
      this.initMap(); //맵 객체 생성
    } else {
      const script = document.createElement('script');
      /* global kakao */
      script.onload = () => kakao.maps.load(this.initMap);
      script.src =
        'http://dapi.kakao.com/v2/maps/sdk.js?autoload=false&appkey=d8b381423317e9a81f56605da1b2a84f';
      document.head.appendChild(script);
    }
  },
  methods: {
    initMap() {
      var container = document.getElementById('map'); //지도를 담을 영역의 DOM 레퍼런스
      var options = {
        center: new kakao.maps.LatLng(this.lat, this.lng), //지도의 중심좌표.
        level: 8, //지도의 레벨(확대, 축소 정도)
      };
      map = new kakao.maps.Map(container, options); //지도 생성 및 객체 리턴
      // 마커를 표시할 위치와 내용을 가지고 있는 객체 배열입니다
      // this.renderMark();
      this.myLocation();
    },
    showMapGu() {
      map.setLevel(7);
      this.hideMarkers(); //기존 랜더링된 마커를 제거한다.
      this.renderMark(); // 마커를 새로 랜더링한다.
    },
    hideMarkers() {
      this.setMarkers(null); //마커배열을 null로
    },
    setMarkers(map) {
      for (var i = 0; i < markers.length; i++) {
        markers[i].setMap(map);
      }
    },
    renderMark() {
      checks = this.checks;
      var gu = this.gu; //this.gu를 지역변수로 받아온다.
      datas = [];
      filteredData = [];
      var lat2; //좌표 임시 변수
      var lng2;
      //json파일을 받아온 데이터를 foreach로
      this.cultures.forEach(function (data) {
        if (data.addr != null) {
          //주소값이 null이 아닌경우
          if (data.addr.includes(gu)) {
            //주소값이 gu(구 변수를 포함하고 있는경우) -> 구 검색
            filteredData.push(data);
            addMarker(data); ////마커에 추가한다.
            lat2 = data.x_coord; //임시변수에 좌표값을 넣는다.
            lng2 = data.y_coord;
          }
        }
      });

      this.datas = datas;
      this.search = '';
      map.panTo(new kakao.maps.LatLng(lat2, lng2)); //중심좌표 이동시킨다.
    },
    movemapto(data) {
      map.setLevel(1);
      map.panTo(new kakao.maps.LatLng(data.x_coord, data.y_coord));
    },

    myLocation() {
      var x, y;
      //임의로 위치를 설정한다.
      x = 37.56692; // X 좌표
      y = 126.97234; // Y좌표
      this.lat = x;
      this.lng = y;
      navigator.geolocation.getCurrentPosition(function (position) {
        // lat = position.coords.latitude; // 위도
        // lng = position.coords.longitude; // 경도
        console.log(position.coords.latitude);
        map.panTo(new kakao.maps.LatLng(x, y));
        // 마커와 인포윈도우를 표시합니다
      });
      map.setLevel(4);
    },
    nearLoc() {
      var loc = map.getCenter();
      my_x = loc.Ma;
      my_y = loc.La;
      this.hideMarkers();
      this.renderLocMark(my_x, my_y);
    },
    renderLocMark(my_x, my_y) {
      checks = this.checks;
      filteredData = [];
      datas = [];
      //json파일을 받아온 데이터를 foreach로
      this.cultures.forEach(function (data) {
        if (data.addr != null) {
          if (
            data.x_coord >= my_x - 0.005 &&
            data.x_coord <= my_x + 0.005 &&
            data.y_coord >= my_y - 0.005 &&
            data.y_coord <= my_y + 0.005
          ) {
            filteredData.push(data);
            addMarker(data); ////마커에 추가한다.
          }
        }
      });

      console.log(
        'center location :[' +
          my_x +
          '/' +
          my_y +
          ']에서 주변시설 총 ' +
          filteredData.length +
          '개 중 ' +
          datas.length +
          '개 출력'
      );
      this.gu = '';
      this.search = '';
      this.datas = datas;
    },
    searchName() {
      checks = this.checks;
      search = this.search;
      console.log(search);
      this.hideMarkers();
      this.renderSearchName();
    },
    renderSearchName() {
      filteredData = [];
      datas = [];
      var lat2 = 0; //좌표 임시 변수
      var lng2 = 0;
      this.cultures.forEach(function (data) {
        if (data.fac_name.includes(search)) {
          filteredData.push(data);
          addMarker(data);
          if (lat2 == 0) lat2 = data.x_coord;
          if (lng2 == 0) lng2 = data.y_coord;
        }
      });
      this.gu = '';
      this.datas = datas;
      map.panTo(new kakao.maps.LatLng(lat2, lng2));
    },
    renderChecked() {
      this.hideMarkers();
      datas = [];
      filteredData.forEach(function (data) {
        addMarker(data);
      });

      this.datas = datas;
    },
  },
};
</script>
<style>
@import 'https://fonts.googleapis.com/css?family=Roboto:300,300i,400,400i,500,500i,700,700i';
@import 'https://fonts.googleapis.com/css?family=Raleway:300,400,700';
@import 'https://fonts.googleapis.com/css?family=Pacifico';
@import 'https://fonts.googleapis.com/css?family=PT+Serif';
@import 'https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css';
@import './resources/css/style.css';
@import './resources/css/typography-default.css';
@import './resources/css/skins/light_blue.css';
@import './resources/css/custom.css';
.map {
  width: 650px;
  height: 550px;
}
</style>
