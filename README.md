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
            //네이버에서 받아온 정보를 저정하기 위한 함수
            $.post("/SNS/Naver/Regist", jsonData, function (rst) {
                console.log(rst);
                if (rst.Check) {
                    opener.snsLogin(rst.Code, function (r) {
                        opener.location.href = "/";
                        window.close();
                    });
                } else {
                    RedAlert(rst.Message);
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
