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
    barColor:"rgb(168,168,168)",
    barBgColor:"rgb(201,201,201)"
};
let canscrollheight = 0; //可滚动高度
let containerheight = 0; //容器高度
let barheight = 0; 
let baroffsetY = 0;
let MvoffsetY = 0;
let mouseY = 0;
let lastBarOffsetY = 0;
let lastScrolltop = 0;

let capture = false;
//DOM
export let container=null;
let bar;
let bartop;
let barbottom;

//刷新滚动条，并计算
function refreshScrollHeight(){
    MvoffsetY = container.scrollTop;
    containerheight = container.clientHeight;
    canscrollheight = container.scrollHeight - container.clientHeight;
    //计算出bar的高度
    barheight = (container.clientHeight / canscrollheight) * container.clientHeight;
    //计算出bar的所在位置
    baroffsetY = (container.scrollTop / canscrollheight) * (container.clientHeight-barheight);
}
//上一次滚动的TOP+鼠标在滚动条上滚动距离，返回最后得到的scrollTOP的位置
function barInctDistance(lastScrolltop, dis){
    return lastScrolltop + ((dis / containerheight) * canscrollheight);
}
function barclick(ev){
    let y = -1;
    if(ev.target==bartop){
        y = ev.offsetY;
    }
    if(ev.target==barbottom){
        y = ev.offsetY + barbottom.offsetTop
    }
    if(y>=0){
        let newy = (y / containerheight)*canscrollheight;
        container.scrollTo(container.scrollLeft,newy);
        refreshScrollHeight();
    }
}
function bardown(ev){
    if(ev.target==bar){
        mouseY = ev.screenY;
        lastBarOffsetY = baroffsetY;
        lastScrolltop = container.scrollTop;
        capture = true;
    }
}
function barup(ev){
    capture = false;
}
//外部更新滚动条方法，调用需要使用bind:this获得实例对象
export function update(){
    refreshScrollHeight();
}

window.addEventListener("mousemove",function(ev){
    if(capture){
        let disY = ev.screenY -mouseY;
        let scY = barInctDistance(lastScrolltop,disY);
        container.scrollTo(container.scrollLeft,scY);
        refreshScrollHeight();
    }
})
window.addEventListener("mouseup",function(ev){
    capture = false;
})

$:if(container){
    console.log(container);
    //刷新滚动条的位置
    refreshScrollHeight();
        
    container.onmousewheel = function(e){
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

<div class="ver-scroll-bar" on:mouseup={barup} on:mousedown={bardown} on:click={barclick} style="top:{MvoffsetY}px;width:{setting.barWidth}px">
    <div class="bar-top" bind:this={bartop} style="background-color: {setting.barBgColor};width:{setting.barWidth}px;height:{baroffsetY}px"></div>
    <div class="bar" bind:this={bar} style="width:{setting.barWidth}px;background-color:{setting.barColor};top:{baroffsetY}px;height:{barheight}px"></div>
    <div class="bar-bottom" bind:this={barbottom} style="background-color: {setting.barBgColor};width:{setting.barWidth}px;top:{baroffsetY+barheight}px"></div>
</div>

<style>
    .ver-scroll-bar{
        user-select: none;
        display: block;
        position: absolute; 
        right:0;
        top:0;
        height: 100%;
    }
    
    .ver-scroll-bar .bar{
        position: absolute;
    }
    .ver-scroll-bar .bar-top{
        user-select: none;
        position: absolute;
        top:0;right:0;left:0;
    }
    .ver-scroll-bar .bar-bottom{
        user-select: none;
        position: absolute;
        left: 0;right: 0;bottom: 0;
    }
</style>
