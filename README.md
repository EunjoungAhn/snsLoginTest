# snsLoginTest
sns Test 관련 정리 코드

# 네이버 로그인 api

### 테스트 환경은 로컬에서 was 서버로 진행하였다.
### 따로 스크립트를 만들어서 값을 index(main화면)으로 가져온다.
### 로그인 팝업 찾을 뛰울 div
```C#
<div id="naver_id_login">네이버</div>

<script src="/Scripts/sns/naver.js"></script>
```

```C#
var naver_id_login = new naver_id_login("YOUR_CLIENT_ID", "http://localhost:27818/SNS/Naver/Callback");
var state = naver_id_login.getUniqState();
naver_id_login.setButton("white", 1, 40);
naver_id_login.setDomain("http://localhost:27818");
naver_id_login.setState(state);
naver_id_login.setPopup();
naver_id_login.init_naver_id_login();
$("#naver_id_login_anchor img").attr("src", "/Content/Images/naver.svg");
```



### CallBack을 처리할 페이지
```C#
@{
    Layout = Global.Layout.Base;
}

@section scriptLayer {
    <script src="//code.jquery.com/jquery-1.12.4.min.js" crossorigin="anonymous"></script>
    <script src="https://static.nid.naver.com/js/naverLogin_implicit-1.0.3.js"></script>
    <script type="text/javascript">
        var naver_id_login = new naver_id_login("YOUR_CLIENT_ID", "http://localhost:27818/SNS/Naver/Callback");
        // 접근 토큰 값 출력
        console.log(naver_id_login.oauthParams.access_token);
        // 네이버 사용자 프로필 조회
        naver_id_login.get_naver_userprofile("naverSignInCallback()");
        // 네이버 사용자 프로필 조회 이후 프로필 정보를 처리할 callback function
        function naverSignInCallback() {
             let jsonData = {
                email: naver_id_login.getProfileData('email'),
                id: naver_id_login.getProfileData('id'),
                name: naver_id_login.getProfileData('name'),
                //로그인시 가져올 수 있는 정보중, 휴대전화와 출생년도는 네이버에서 제공하는 Js sdk를 받아서 적용해야 한다.
                //mobile: naver_id_login.getProfileData('mobile'),
                birthday : naver_id_login.getProfileData('birthday'),
                //birth : naver_id_login.getProfileData('birth'),
                gender : naver_id_login.getProfileData('gender')
            };
            console.log("jsonData:", jsonData);
           //네이버 로그인 버튼 클릭시 회원 여부 확인
        $.post("/SNS/Naver/ID/Check", jsonData, function (rst) {
            console.log(rst);
            if (rst.check) {
                opener.location.href = "/";
                window.close();
            } else {
                alert("회원이 아닙니다. 회원가입을 진행해 주세요.");
                window.close();
                opener.location.href = `/Member/join?email=${jsonData.email}&name=${jsonData.name}&id=${jsonData.id}`;
            }
        });
            
        }
    </script>
}
```


### 컨트롤러
```C#
        [HttpPost]
        [Route("SNS/Naver/Regist")]
        public JsonResult NaverRegist(string id, string email, string name = "")
        {
            var result = new ReturnValue();

            if (!string.IsNullOrWhiteSpace(id))
            {
                result = this.Db.SNSNaverJoin(id, email, name);
            }
            else
            {
                result.Error("잘못된 요청입니다.");
            }

            return Json(result);
        }
```

# 카카오 로그인 api

### 로그인 버튼 추가 및 버튼 모양 설정
```C#
<a href="javascript:fnLoginFromKakao();"><img src="/Content/Images/kakaotalk.svg" alt="카카오톡 로그인" /></a>
```

### api 사용을 위한 sdk 스크립트 추가
```C#
<script src="https://developers.kakao.com/sdk/js/kakao.min.js"></script>
```

### 스크립트
```C#

var kakaoKey = '카카오 api key';
var kakaoAuth = null;

$(document).ready(function () {
    Kakao.init(kakaoKey); //발급받은 키 중 javascript키를 사용해준다.
});
//카카오로그인
function fnLoginFromKakao() {
    Kakao.Auth.login({
        success: function (authObj) {
            kakaoAuth = authObj;
            fnKakaoUserInfoGet();
        },
        fail: function (err) {
            alert(JSON.stringify(err));
        },
    });
};
//카카오 로그인 사용자 정보 가져와서 회원 여부 확인 후, 아니면 가입화면으로
function fnKakaoUserInfoGet() {
    Kakao.API.request({
        url: '/v2/user/me',
        success: function (res) {
            let jsonData = { UserID: res.id, Email: res.kakao_account.email, UserName: res.kakao_account.profile.nickname };
            console.log("jsonData", jsonData)
            $.post("/SNS/Kakao/ID/Check", jsonData, function (rst) {
                console.log("rst", rst)
                if (rst.check) {
                    location.href = "/";
                } else {
                    alert("회원이 아닙니다. 회원가입을 진행해 주세요.");
                    location.href = `/Member/join?email=${jsonData.Email}&name=${jsonData.UserName}&id=${jsonData.UserID}&joinType=Kakao`;
                }
            });
        },
        fail: function (error) {
            alert(
                'login success, but failed to request user information: ' +
                JSON.stringify(error)
            );
        },
    });
};
```

# 구글 로그인 api
### 로그인 버튼 활성화를 위한 meta 설정
```C#
<meta name="google-signin-client_id" content="OAuth2.0 클라이언트 ID">
```

### 로그인 버튼
```C#
<div class="g-signin2"></div>
```

### api 사용을 위한 sdk 스크립트 추가
```C#
<script src="https://apis.google.com/js/platform.js" async defer></script>
```

### 로그인 버튼 에러
아래의 에러는 Google Auth API 초기화 실패(클라이언트에 타사 쿠키가 활성화되지 않은 경우 발생)하는 것이다.
error: 'idpiframe_initialization_failed', details: 'R'

### 이유&  해결방안

개인 정보 보호를 위해 "타사 쿠키 및 사이트 데이터 차단"을 설정해야한다. </br>
사용자가 타사 세션 스토리지를 사용하지 않도록 설정되어 있었기 때문에 클라이언트가 작동하지 않은 것이었다.</br>
</br>
[해결방안]
Google Chrome 브라우저 설정 > 개인정보 및 보안 > 쿠키 및 기타 사이트 데이터 > </br>
모든 쿠기 허용 or 하단에 "쿠키를 언제든지 사용할 수 있는 사이트" 에 [*.]google.com 주소 추가!

### 
```C#
function onSignIn(googleUser) {
    // Useful data for your client-side scripts:
    var profile = googleUser.getBasicProfile();
    /*
    console.log("ID: " + profile.getId()); // Don't send this directly to your server!
    console.log('Full Name: ' + profile.getName());
    console.log("Email: " + profile.getEmail());

    // The ID token you need to pass to your backend:
    var id_token = googleUser.getAuthResponse().id_token;
    console.log("ID Token: " + id_token);
    */

    let jsonData = { UserID: profile.getId(), Email: profile.getEmail(), UserName: profile.getName() };
    console.log("jsonData", jsonData)
    $.post("/SNS/google/ID/Check", jsonData, function (rst) {
        console.log("rst", rst)
        if (rst.check) {
            location.href = "/";
        } else {
            alert("LSK 회원이 아닙니다. 회원가입을 진행해 주세요.");
            location.href = `/Member/join?email=${jsonData.Email}&name=${jsonData.UserName}&id=${jsonData.UserID}&joinType=Google`;
        }
    });
};

function onFailure(error) {
    console.log(error);
}

//구글 버튼 커스톰 함수 
function renderButton() {
    gapi.signin2.render('googleLogin', {
        'scope': 'profile email',
        'width': 240,
        'height': 40,
        'longtitle': true,
        'onsuccess': onSignIn,
        'onfailure': onFailure
    });
}
```

### 커스텀 버튼 함수를 사용할때는 api 뒤에 ? 와 함께 해당 함수명을 적으면 됩니다.
```C#
<script src="https://apis.google.com/js/platform.js?onload=renderButton" async defer></script>
```
