```javascript
<script type="text/javascript">
var url = 'http://localhost:8080/';
function jsonp(obj,time,url){
    //基本类型
    url+=url.indexOf('?')===-1?'?':'&';
    for(let item in obj){
        url+=encodeURIComponent(item)+'='+encodeURIComponent(obj[item])+'&';
    }
    var callBackName =('_jsonp'+Math.random()).replace('.','')
    url+='callback='+callBackName;
    var scriptEle = document.createElement('script');
    scriptEle.src = url;
    document.head.appendChild(scriptEle);
    window[callBackName] = function(data){
       	//这里是对数据进行处理
        window.clearTimeout(timer);//清除定时器
        document.head.removeChild(scriptEle);
        window[callBackName] = null;//把回调函数解除引用
    }
    //超时处理
    var timer = window.setTimeout(function(){
        document.head.removeChild(scriptEle);//移除script标签
		window[callBackName] = null;//把回调函数解除引用
    },time)
}
jsonp({name:'dd'},5000,url)
</script>
```