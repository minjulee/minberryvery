---
layout: post
title:  "html5 & flash crossbrowsing script"
date:   2016-01-19 18:36:19 +0900
categories: javascript
---

사용자에게 동영상을 제공하기 위해 여러가지 방법이 쓰이고 있다. 그중 video, flash를 지원하는 것이 보편적인 방법이며
다양한 브라우저와 ie 버전에 대응하는 PLAY, STOP 기능 및 현재 시간, 종료 시간을 얻는 스크립트를 모듈화했다.



## Javascript Source
{% highlight javascript linenos %}
    // Editor : LMJ
    // Date   : 2016-01-12
    // Desc   : 모든 브라우저에서 동영상을 볼 수 있다.
    // Module Init
    var tgm = tgm || {};
    // Video Module
    tgm.video = (function(){
        // Private Area
        var movie;
        // Properties for flash object
        var flash = function(obj){
            Stop = function(){
                obj.StopPlay();
            };
            Play = function(){
                obj.Play();
            };
            GetCurrentTime = function(){
                return obj.TCurrentFrame("_root"); 
            };
            GetTotalTime = function() { 
                return obj.TGetProperty('/', 5); 
            };

            return this;
        };
        // properties for video tag
        var video = function(obj){
            Stop = function(){
                obj.pause();
            };
            Play = function(){
                obj.play();
            };
            GetCurrentTime = function(){
                return obj.currentTime;
            };
            GetTotalTime = function(){
                return obj.duration;
            };
            return this;
        };
        // flash or video check
        var getVideoObject = function(pVideoName){
            var obj;
            
            if(!!document.createElement('video').canPlayType){       
                obj = document.getElementsByTagName("video");
                for(var i = 0; i < obj.length; i++){
                    var name = obj[i].getAttribute("name");
                    if(name === pVideoName){
                        movie = video(obj[i]);
                    }
                }
            }else{                    
                if(window.document[pVideoName]){
                    obj = window.document[pVideoName];
                }
                if(navigator.appName.indexOf("Microsoft Internet") === -1){
                    if(document.embeds && document.embeds[pVideoName]){
                        obj = document.embeds[pVideoName]
                    }
                }else{
                    var objs = document.getElementsByTagName("object");
                    for(var i = 0; i < obj.length; i++){
                        var name = objs[i].getAttribute("name");
                        if(name === pVideoName){
                            obj = objs[i];
                        }
                    }
                }
                movie = flash(obj);    
            }
        };

        return {
            SetName : function(pVideoName){
                getVideoObject(pVideoName);
            },
            Stop : function(){
                movie.Stop();
            },
            Play : function(){
                movie.Play();
            },
            GetCurrentTime : function(){
                return movie.GetCurrentTime();
            },
            GetTotalTime : function(){
                return movie.GetTotalTime();
            }
        };
    })();
    // 사용예제
    $(document).ready(function(){
        // 세팅
        tgm.video.SetName("myFlashMovie");
        // 인터벌 객체 생성
        var interval;
        // 이벤트 바인딩
        $("#play").click(function(){
            tgm.video.Play();
            // 클리어 
            if(interval){
                clearInterval(interval);    
            }
            // 초기화
            interval = setInterval("test()", 200);
        });          
        $("#stop").click(function(){                
            tgm.video.Stop();
            // 클리어
            clearInterval(interval);
        });          
    });
    // 현재 위치 / 종료 위치 표시
    function test(){
        $("#spanT1").text(tgm.video.GetCurrentTime());
        $("#spanT2").text(tgm.video.GetTotalTime());
    }

{% endhighlight %}
## Css Source
{% highlight css linenos %}

    <style>
        .video_wrapper{
            position:relative;
            width:100%;
            height:600px;
            border:1px solid #bbb;
        }
        .video_btn{
            position:absolute;
            top:50%;
            left:50%;
            width:50px;
            height:50px;
            margin-top:-25px;
            margin-left: -25px;
            background-color:red;
            display: none;
        }
    </style>

{% endhighlight %}
## HTML Source
{% highlight html linenos %}

    <p>
        현재<span id="spanT1">0</span>&nbsp;&nbsp;-&nbsp;&nbsp;종료<span id="spanT2">0</span>
    </p>
    <div class="video_wrapper">
        <div class="video_btn">종료 후 이벤트?</div>
        <!-- Flash(swf) Test -->
       <video name="myFlashMovie" width="640" height="360" autoplay="autoplay" preload="auto">
            <source src="http://movie.woogos.com/750/20151008_5.mp4" type="video/mp4"/>
            <object name="myFlashMovie" width="481" height="86" classid="clsid:D27CDB6E-AE6D-11cf-96B8-444553540000" codebase="http://download.macromedia.com/pub/shockwave/cabs/flash/swflash.cab#version=6,0,29,0">
                <param name="movie" value="choudanse7.swf" />
                <param name="wmode" value="transparent" />
                <param name="quality" value="high" />
                <param name="loop" value="false" />
                <embed src="choudanse7.swf" name="myFlashMovie" wmode="transparent" loop="false" quality="high" width="481" height="86"
                       type="application/x-shockwave-flash" pluginspage="http://www.macromedia.com/shockwave/download/index.cgi?P1_Prod_Version=ShockwaveFlash"> </embed >
            </object >
        </video>
    </div>

    <form name="controller" method="POST">
        <div style="text-align: center">
            <input type="button" value="Play" name="Play" id="play" />
            <input type="button" value="Stop" name="Stop" id="stop" />
            <br>
        </div>
    </form>

{% endhighlight %}
## 테스트 환경

VMWare(windowsXP ie7, windowsXP ie8, windows7 ie9),
windows10 ie11, chrome(v 47), firefox(v 43)