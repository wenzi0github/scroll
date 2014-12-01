    在现在更加追求页面加载速度和用户体验的情况下，页面的滚动事件使用的越来越多。通常我们使用滚动事件主要做的事情主要有：

* ajax异步加载，加快页面首次加载的速度
* 懒加载（或延迟加载）：先把HTML元素放到textarea标签中，或把img的链接先放到一个字段里，页面滚动到某个位置时才进行开始加载
* 顶部导航或侧边导航的焦点跟踪
* 侧边数字导航的跟踪（例如百度经验）
* “回到顶部”功能
 

 　　本项目中用滚动事件实现了：懒加载，顶部导航焦点追踪，侧边数字导航和“回到顶部”功能，不过demo中没有实现异步加载的功能，其实懒加载和异步加载的原理差不多，只不过一个是先把数据请求了只是不加载，一个是滚动到位置了才用ajax请求数据。
在这里我讲一下这些功能的实现方式：

　　1. 顶部导航的焦点追踪

　　焦点的追踪，顾名思义：当我们移动到哪个区域时，焦点就移动到哪个导航元素上，指示我们当前查看的是哪个区域。当然，这个功能的前提是我们需要知道每个元素距内容顶部的高度。
```javascript
// 获取每个item距顶部的高度,$item为区域的综合，listTop用来存储每个区域距顶部的高度
$item.each(function(index, el) {
    listTop.push($(el).offset().top);
});
```

　　我们从demo中也能看到，当aaaa区域到达顶部时，导航的position变成fixed，然后开始跟踪；滚动条向上滑动，aaaa离开顶部时，导航重新变回原来的样式。其实，这只是我们看到是这个样子，导航的样式切换来切换去。但实际上我们并不是这样实现的，实际中这是两个导航（O(∩_∩)O~），只不过让其中一个导航（称为A）不动，另一个导航（称为B）显示隐藏而已，导航B填充导航A的内容即可。在获取了这些item距顶部的高度后，那么我们就在滚动事件中判断，滚动条的高度是否超过了第一个item的高度，如果超过导航B显示即可，否则导航B隐藏（代码中.navfix即导航B，winTop为滚动条的高度）。

// 是否显示顶部导航
winTop < listTop[0] ? $(".navfix").hide() : $(".navfix").show();
 

　　现在重点来了，导航B显示出来了，那么就需要当前所在的区域和焦点对应上：刚才我们已经获取到每个区域的高度了，现在我们就计算一下滚动条的高度在哪个区间（编号K），计算出区间后，我们就可以给哪个导航元素相应的样式了：

winTop<listTop[0]  ： 不在任何区域
winTop>=listTop[0] && winTop<listTop[1] : 在区域aaaa
winTop>=listTop[1] && winTop<listTop[2] : 在区域bbbb
winTop>=listTop[2] && winTop<listTop[3] : 在区域ccccc
inTop>=listTop[3] : 在区域dddd

// 检测所在区域
for (; i < t; i++) {
    if ( winTop > listTop[t-1] ) {
        k = t-1;
        break;
    }else if ( winTop>=listTop[i-1] && winTop < listTop[i] ) {
        k = i-1;
        break;
    }
}

// 顶部导航效果
if( k > -1 ){
    $li.removeClass('hover');
    $li.eq(k).addClass('hover');
}
 

　　k默认的是-1，即不在任何区域，若k>-1即肯定处在某个区域内，先清除导航中所有元素的样式，然后再指定样式

 

　　2. 侧边数字导航

　　其实侧边数字导航与顶部导航实现的原理一样：数字侧边栏也是有两个（跟着区域移动的数字导航A，固定导航B），当某个数字跟着区域移动时，导航B中相应的数字隐藏；当数字到达顶部时，导航A中的数字隐藏，导航B中的数字显示；即使区域的数字到达顶部不再移动，那也不能立即变成灰色，应当还是绿色，只有该区域超过窗口上边框才能变成蓝色。这就形成了我们现在看到的效果。

　　这里的重点是计算出什么时候隐藏导航A中的数字，显示导航B中的数字，而且导航B的数字显示什么颜色：每次滚动时，都首先让导航A中的数字显示，导航B的数字隐藏，然后计算每个区域所在的位置，如果某个区域距顶部的高度与滚动条的高度小于了导航B的数字的高度，就说明导航A中的数字该隐藏，导航B的数字该显示了；那显示的数字呈现什么颜色呢，刚才我们计算出了当前所在区域的编号K，那么区域编号小于编号K都是已经看过的，就显示灰色，否则就是正在看或者没看的区域就显示绿色。

// 侧边数字导航
$item.find(".item-icon").show();    // 跟着区域移动的数字
$step_a.css('visibility', 'hidden');// 固定导航的数字
for(i=0; i<t; i++){
    if(listTop[i]-winTop<i*32+35){
        $item.eq(i).find(".item-icon").hide();
        $step_a.eq(i).css({'visibility':'visible', 'background-color': (i<k?'#888':'#008B00') });
    }
}


　　3. 懒加载

　　通常加载DOM元素时需要对页面进行渲染，耗费时间，那么我们就先把这些DOM元素存储起来，等需要加载的时候再去加载，用来加快页面初始的加载；img图片同理。

// 懒加载底部内容
if( $copyright.attr("loaded")!="loaded" && (winTop+800 > copyTop)){
    var text = $copyright.find('textarea').val();
    $copyright.html(text);

    $copyright.attr("loaded", "loaded");
}
 

 

　　图片的路径我们可以先放到一个字段中，比如data-src，等需要加载该图片时，则从data-src中取出该值并赋值给src，然后请求图片.

 

　　4. “回到顶部”功能

　　“回到顶部”功能，即将scrollTop的值设置为0的过程，如果需要缓冲效果，那么就给它一个缓冲时间

// 回到顶部
$("#backtotop").on('click', function(event) {
    event = event || window.event;

    var winTop = $(window).scrollTop();
    $('body').animate({scrollTop:0}, winTop/4);

    event.preventDefault();
});
 

　　5. 其他

　　当然，这里还有一些东西没有提到，不过也很重要，比如如何固定滚动条不能移动，回到顶部里的小三角的制作等等；
