# snsLoginTest
sns Test 관련 정리 코드

#네이버 로그인 api

###테스트 환경은 로컬에서 was 서버로 진행하였다.
###따로 스크립트를 만들어서 값을 index(main화면)으로 가져온다.
###로그인 팝업 찾을 뛰울 div
<div id="naver_id_login">네이버</div>

<script src="/Scripts/sns/naver.js"></script>

var naver_id_login = new naver_id_login("YOUR_CLIENT_ID", "http://localhost:27818/SNS/Naver/Callback");
var state = naver_id_login.getUniqState();
naver_id_login.setButton("white", 1, 40);
naver_id_login.setDomain("http://localhost:27818");
naver_id_login.setState(state);
naver_id_login.setPopup();
naver_id_login.init_naver_id_login();
$("#naver_id_login_anchor img").attr("src", "/Content/Images/naver.svg");




###CallBack을 처리할 페이지
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
                name: naver_id_login.getProfileData('age')
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
