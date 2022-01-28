# MIXUULS.github.io
黑珍

<!DOCTYPE html >
< html  lang =" en " >

<头>
    <元 字符集=" UTF-8 "/>
    < meta  name =" viewport " content =" width=device-width, initial-scale=1.0 "/>
    < meta  http-equiv =" X-UA-Compatible " content =" ie=edge "/>
    < title >悦听播放器</ title >
    <!-- 样式 -->
    <链接 rel =“样式表” href =“ ./css/index.css ” >
</头>

<身体>
< div 类="换行" >
    <!-- 播放器主体区域 -->
    < div  class =" play_wrap " id ="播放器" >
        < div 类="搜索栏" >
            < img  src ="图像/player_title.png"alt = " "/>
            <!-- 搜索歌曲 -->
            <输入 类型="文本"自动完成="关闭" v-model ="查询" @keyup.enter =" searchMusic "/>
        </ div >
        < div 类=" center_con " >
            <!-- 搜索歌曲列表 -->
            < div 类=' song_wrapper ' >
                < ul 类=“歌曲列表” >
                    < li  v-for ="音乐列表中的项目" >
                        <a href="javascript:;"@click="playMusic(item.id)"> </a> _ _  _ _ _ _ _ _ _ _ _
                        < b > {{item.name}} </ b >
                        < span  v-if =" item.mvid!=0 " @click =" playMovie(item.mvid) " > < i > </ i > </ span >
                    </ li >
                </ ul >
                < img  src =" images/line.png " class =" switch_btn " alt ="" >
            </ div >
            <!-- 歌曲信息容器 -->
            < div  class =" player_con " :class =" {playing:isPlaying} " >
                < img  src ="图像/player_bar.png "类=" play_bar "/>
                <!-- 黑胶碟片 -->
                < img  src =" images/disc.png " class =" disc autoRotate "/>
                < img  :src =" musicCoverUrl "类="封面自动旋转"/>
            </ div >
            <!-- 容器评论 -->
            < div 类=" comment_wrapper " >
                < h5  class =' title ' >热门留言</ h5 >
                < div 类=' comment_list ' >
                    < dl  v-for =" hotComments 中的项目" >
                        < dt > < img  :src =" item.user.avatarUrl " alt ="" > </ dt >
                        < dd  class =" name " > {{item.user.nickname}} </ dd >
                        < dd 类=“详细” >
                            {{item.content}}
                        </ dd >
                    </ dl >
                </ div >
                < img  src =" images/line.png "类=" right_line " >
            </ div >
        </ div >
        < div 类=" audio_con " >
            < audio  ref =' audio ' @play =" play " @ pause =" pause " :src =" musicUrl "控制自动播放循环  
                   类=" myaudio " > </音频>
        </ div >
        < div 类=" video_con " v-show =" showVideo " >
            <视频 :src =" mvUrl "控件="控件" > </视频>
            < div  class =" mask " @ click =" closeMv " > </ div >
        </ div >
    </ div >
</ div >
<开发版本，包含有帮助的警告-->
< script  src =" https://cdn.jsdelivr.net/npm/vue/dist/vue.js " > </ script >
<!-- 官网提供的 axios 在线地址 -->
< script  src =" https://unpkg.com/axios/dist/axios.min.js " > </ script >

<脚本>
    var  vm  = 新的 Vue ( {
        埃尔：“#player” ，
        数据：{
            查询：“” ，
            音乐列表：[ ] ，
            音乐网址：“” ，
            音乐封面网址：“” ，
            热门评论: [ ] ,
            正在播放：假，
            显示视频：假，
            mvUrl : "" ,


        } ,
        方法：{
            搜索音乐( )  {
                控制台。日志（this.query ）_ _
                var  that  =  this ;
                斧头。获取( "  https://autumnfish.cn/search?keywords="+that.query )  。_ _ _ 那么（
                    功能 （响应） {
                        // console.log(response);
                        那个。音乐列表 = 响应。数据。结果。歌曲；
                    } ,
                    功能 （错误） {
                    }
                ) ;
            } ,
            播放音乐（音乐 ID ） {
                var  that  =  this ;
                // 获取歌曲
                斧头。获取( "  https://autumnfish.cn/song/url?id="+musicId ) 。_  然后（函数（响应）{  
                        那个。musicUrl  = 响应。数据。数据[ 0 ] 。网址；
                    } ,
                    功能 （错误） {
                    } ) ;

                // 歌曲详情获取
                斧头。获取( "  https://autumnfish.cn/song/detail?ids="+musicId ) 。_  那么（
                    功能 （响应） {
                        // console.log(response.data.songs[0].al.picUrl);
                        那个。musicCoverUrl  = 响应。数据。歌曲[ 0 ] 。人_ 图片网址;
                    } ,
                    功能 （错误） {
                    }
                ) ;

                // 歌曲评论获取
                斧头。获取( "  https://autumnfish.cn/comment/hot?type=0&id="+musicId ) 。_  那么（
                    功能 （响应） {
                        // console.log(response);
                        控制台。日志（响应。数据。热评论）；
                        那个。热门评论 = 响应。数据。热门评论；
                    } ,
                    功能 （错误） {
                    }
                ) ;
            } ,
            // 歌曲播放
            播放：函数 （） {
                // console.log("play");
                这个。isPlaying  =  true ;
            } ,
            // 视频暂停
            暂停：函数 （） {
                // console.log("暂停");
                这个。isPlaying  =  false ;
            } ,
            播放电影( mvid ) { 
                var  that  =  this ;
                那个。显示视频 = 真；
                斧头。获取( "  https://autumnfish.cn/mv/url?id="+mvid ) 。_  那么（
                    功能 （响应） {
                        // console.log(response);
                        控制台。日志（响应。数据。数据。网址）；
                        那个。$refs 。音频。暂停( )
                        那个。mvUrl  = 响应。数据。数据。网址；
                    } ,
                    功能 （错误） {
                    }
                ) ;
            } ,
            关闭MV ( )  {
                这个。显示视频 = 假；
                这个。mvUrl  =  "" ;
            }
        }
    } )

</脚本>
</正文>

</ html 
