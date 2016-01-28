# ipeiyin

  前天看到空间里有人在玩给视频里的人说的话配音。有两种，一种其实就是卡拉OK的形式，把话筒交给你；另一种是给电影里的人物配音，就是说台词。想想近期有给你一段音频要你自己表演对口型的，还有给你视频要你自己配音的。
  我也玩了一下，录完之后分享到微信朋友圈，结果我点击分享按钮之后，这软件提示我“发送取消”。我心想我明明点的“发送”啊，怎么就成“取消”了。好行，取消就取消吧，我就再发送一次，结果还是“发送取消”，反正前后总共云雨了三次之后，我放弃了，然后就去吃饭了。结果回来一看，咦，怎么朋友圈里这么多条响应？</br>
<img src = http://raw.github.com/caiqiqi/ipeiyin/master/img/1.png width="50%" height="50%" /> </br>
然后我把自己的朋友圈的“相册”打开发现已经发布了三条，由于后来又被我删除了，这里只剩下两条。
<img src = http://raw.github.com/caiqiqi/ipeiyin/master/img/2.png width="50%" height="50%" /> </br>
我寻思原来您玩延迟呢！
  然后我就想看看里面代码到底怎么写的。话说上次玩“唱吧”也是的，第一次玩就发现一个bug，就是当你打开一个你之前已经打开过的曲目时，而且是同一个版本的话，即，该版本的缓存已经在你本地了，比如我打开上次唱过的“Lose yourself”，因为这首歌有几个版本，去确认了是刚才打开的那个版本之后，当我再点击那个曲目时，就进不去了，而打开其他任何我没打开过的曲目都挺好的。。。后来我反馈给了“唱吧”新版本解决没有。
PS：这么简单的一个bug也没发现。。。（难道因为众ROM难调，是我机型问题？）
  好，闲话不多扯，看看里面代码到底怎么搞的。首先当然是先在手机里观察，这里应用的底部有四个标签页（TabHost）：“配音”“小组”“榜单”“我”。
<img src = http://raw.github.com/caiqiqi/ipeiyin/master/img/3.png width="50%" height="50%" /> </br>
 然后audio,group,rank,me…几个单词就出现在我脑海里，在猜想他会怎么命名呢？
好，首先当然是打开配置文件（`AndroidManifest.xml`）咯。搜索**android.intent.action.MAIN** **android.intent.category.LAUNCHER**的“活动”（Activity）。好找到了，**com.ishowedu.peiyin.login.SplashActivity** </br>
嗯，这里不是一个主界面，应该是一个欢迎界面。好像一般都这个路数哈，先是一张欢迎的图，然后如果没有账号在本地登录过，即本地没有你的SharedPreference的话，就进入登录界面；如果你已经登录过了，那么直接进入主界面。</br>
打开这个Activity，注意到这个Activity没有直接继承Activity类，而是</br>

<img src = http://raw.github.com/caiqiqi/ipeiyin/master/img/4.png width="50%" height="50%" /> </br>
好了，先不去研究BaseActivity2是个什么玩意，先搜索onCreate()吧，结果居然没有，啊，我还是太年轻，原来您不重写这个啊？行，那就搜startActivity()吧，您总得跳转的吧。搜到一个叫onLoadFinished()的一个方法，可能是BaseActivity2的吧，啊，不管了，好，找到线索了。 
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/5.png) 
看到经过判断后，它会跳转到MainActivity。然后再搜MainActivity的onCreate()方法，这次总算有了吧
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/6.png) 
看到一个init()，看下面也没有重要的点了，关键应该在init()里面。
结过看了init()之后，发现我想错了，我看init()干嘛，我现在底下有四个TabHost，我应该找TabHost啊！于是搜索TabHost之后发现了这个
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/7.png) 
哈哈，这不就是了，home,group,rank/show,me这些东西。
将GroupHomeWrapperFragment与“小组”选项卡配对，RankFragment2与“榜单”配对，MeSpaceFragment与“我”选项卡配对之后，就只剩下HomeFragment了，好吧，那只能是你了。所以我目前所在的“配音”选项卡应该是HomeFragment。
我要找到我的地盘，然后打开我先前配的音当然要到MeSpaceFragment里面去看啊。
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/8.png) 
看到这个页面有很多选项，可能是Button或者什么View之类的。看看全局变量吧，找到很多TextView
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/9.png)

想到这个界面里的各种词“粉丝”“访客”“相册”什么的，然后什么fans,visitor,album,friend之类的东西就出来了，看看变量名，恩，应该就是你们这几个家伙了。然后我发现到现在为止我已经看到好多次dubbing这个词了，这词啥意思啊，没见过。查了查，哦，原来是“配音”的意思。那“我的配音”这个按钮应该对应tvMydub这个了。那就该去找点击之后会发生什么啊，于是找onClick()。在onClick()中有很多case，对应着这个界面中的各个View控件，再找startActivity()。好像没有什么信息需要返回的，应该不是startActivityForResults()。在众多的startActivity()中找吧。 
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/10.png) 
看到一个DubbingListActivity，这个应该是表示“配音列表”吧。但是startActivity()中的参数怎么不是直接指定的Intent呢？猜想createIntent()应该是返回一个Intent，去里面瞄了瞄，恩，是的，好，这我就放心了，然后就进入了DubbingListActivity，这时候我们就来到这里了
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/11.png) 
捣鼓了半天，在它的onCreate()中发现这两个方法initActionBar()和initFragment()
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/12.png) 
 经过验证发现，应该上面是ActionBar，下面是Fragment。
然后就找onClick()，找了半天发现没找对地方，后面想了想，这活应该交给它的Fragment，于是在initFragment()中找到了对应的DubbingListFragment。
又是一番好找（第一次玩，没经验啊），在DubbingListFragment中找到了两个响应点击事件的监听器（在那个有视频截图的图片上长按，发现有响应的对话框出现，对应的，猜想应该有一个对长按响应的监听器）。
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/13-1.png) 
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/13-2.png)
长按的就不看了吧，看OnItemClickListener这个的onItemClick()吧，找到关键代码
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/14.png)
发现一个新的HotRankInfoActivity。看名字不像是这个界面的感觉啊
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/15.png)
就因为这个名字，没有想通，在别的地方又找了好久（所以还是没经验啊）。
最后还是根据在这个界面中点击“评论”和“分享”按钮时，系统弹出的Toast“当前为本地视频，请上传后重试”这个线索，发现还只能是这里了。
//当然，同时我也因此找到了与“评论”和“分享”按钮相关的代码区域。
好了，再来，再找它的onCreate()
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/16.png)
再看initFragment()这个方法
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/17.png)
（先前对Fragment不怎么熟，临时抱佛脚，查了下官方文档才知道怎么用的。）于是，我们又要跳到HotRankInfoFragment里去瞧一瞧了。
照例，还是在onClick()中找，发现switch里有几个case，应该是对应的“评论”“我要配音”“分享”这几个按钮。
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/18.png)
看到698行，“分享”按钮下面，又是对操作者是否是本地用户进行了判断。
最后关键的是707行（开始以为709行会干什么重要的事咯，后来发现原来只是给新打开的Activity加一些特效），将各种乱七八糟的参数传给Intent后，进入到ShareActivity，也就是这个界面：
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/19.png)
在ShareActivity中，又是通过对onClick()的查看发现了各个case
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/20.png)
发现对分享到“微信好友”和“微信朋友圈”有一条两者都需要走的路。

![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/21.png)

都是先检测“微信”的版本，确认该版本可以分享之后，再通过对weChatShare()传递不同的参数，以区分“发送给微信好友”还是“分享到朋友圈”这两种情况。
在weChatShare()中，通过调用腾讯的微信SDK中的API，将这个“配音”的各种信息（视频的url，封面图片，描述语等）交给微信的Activity来完成。
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/22.png)
最后，我想不通的是我点击了“发送”按钮之后，这个应用给我的提示居然是莫名其妙的“发送取消”
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/23.png)
我通过“发送取消”这个字符串找了半天也没找到是在哪里出现的。。。
最后我在smali目录下搜索所有文件中是否有包含“0x7f0d0099”的字符串出现，结果开始很让人兴奋
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/24.png)
但后来发现它出现在微信的API中的一个回调方法onResp(BaseReq paramBaseReq)中，至于怎么微信是怎么处理“发送”按钮的，我就不得而知了。所以我们的探索到这里也只能结束了。
最后总结一下，第一次玩逆向，太年轻，走了很多弯路，其实应该直接adb连接手机，在DDMS里面直接观察就可以发现当前处在哪个Activity，调用了哪个方法。
最后顺便瞄了一下DDMS，突然有个”xiaomi”的Tag，我好奇，我寻思这谁啊？我又没用小米的手机，怎么又这个东西，后来找到原来是bilibili的pushservice…
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/25.png)
一怒之下，将其禁用。随后又想起各种无聊的推送消息，一一禁用。
![Image text](http://raw.github.com/caiqiqi/ipeiyin/master/img/26.png)

