<script>
// 说明，父组件必须带有的样式是：{ position: 【absolute，fixed】; overflow: hidden; } 
/*
  例子：
  App.svelte:
    <script> 	
      import VerScrollBar from './VerScrollBar.svelte'; 
      let mainObj; 
    </script>
    <div class="test" bind:this={mainObj}>
      <VerScrollBar bind:container={mainObj} />
    </div>
*/
import { onMount } from 'svelte';
export let setting = {
    barWidth:17,
    barColor:"gray",
    barBgColor:"rgb(201,201,201)"
};
export let container;
let canscrollheight = 0;
let barheight = 0;
let baroffsetY = 0;
let MvoffsetY = 0;

function refreshScrollHeight(){
    MvoffsetY = container.scrollTop;
    canscrollheight = container.scrollHeight - container.clientHeight;
    //计算出bar的高度
    barheight = (container.clientHeight / canscrollheight) * container.clientHeight;
    //计算出bar的所在位置
    baroffsetY = (container.scrollTop / canscrollheight) * (container.clientHeight-barheight);
}

$:if(container){
    //可滚动高度
    refreshScrollHeight();
        
    container.onmousewheel = function(e){
        console.log(e)
        if(e.wheelDeltaY<0){
            //向下滑
            container.scrollTo(container.scrollLeft,container.scrollTop -(e.wheelDeltaY) );
        }else{
            //向上滑
            container.scrollTo(container.scrollLeft,container.scrollTop -(e.wheelDeltaY) );
        }
        refreshScrollHeight();
    }
}
</script>

<div class="ver-scroll-bar" style="top:{MvoffsetY}px;width:{setting.barWidth}px">
    <div class="bar-top" style="background-color: {setting.barBgColor};width:{setting.barWidth}px;height:{baroffsetY}px"></div>
    <div class="bar" style="width:{setting.barWidth}px;background-color:{setting.barColor};top:{baroffsetY}px;height:{barheight}px"></div>
    <div class="bar-bottom" style="background-color: {setting.barBgColor};width:{setting.barWidth}px;top:{baroffsetY+barheight}px"></div>
</div>

<style>
    .ver-scroll-bar{
        display: block;
        position: absolute; 
        right:0;
        top:0;
        height: 100%;
    }
    
    .ver-scroll-bar .bar{
        position: absolute;
        background-color: gray;
    }
    .ver-scroll-bar .bar-top{
        position: absolute;
        top:0;right:0;left:0;
    }
    .ver-scroll-bar .bar-bottom{
        position: absolute;
        left: 0;right: 0;bottom: 0;
    }
</style>
