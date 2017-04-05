○ 浏览器的缓存机制

对于web应用来说,浏览器缓存是提升页面性能同时减少服务器压力的利器.

● 浏览器缓存的类型

有两种,强缓存和协商缓存

1.强缓存: 不会向服务器发送请求,直接从缓存中读取资源,在chrome控制台的network选项中可以看到该请求返回200的状态码,并且size
    显示from disk cache或from memory cache;

2.协商缓存: 向服务器发送请求,服务器会根据这个请求的request header的一些参数来判断是否命中协商缓存,如果命中,则返回304状态
    码并带上新的response header通知浏览器从缓存中读取资源;

3.两者的共同点是,都是从客户端缓存中读取资源;区别是强缓存不会发请求,协商缓存会发请求.

● 缓存有关的header

◇ 强缓存

    ①Expires: response header里的过期时间,浏览器再次加载资源时,如果在这个过期时间内,则命中强缓存.

    ②Cache-Control: 当值设置为max-age=300时,则代表在这个请求正确返回的5分钟内,再次加载该资源,就会命中强缓存.

    (图片)

    Expires和Cache-Control作用是差不多的,区别就在于Expires是http1.0的产物,Cache-Control是http1.1的产物,两者同时
    存在的话Cache-Control的优先级更高(现阶段Expires的存在只是一种兼容性的写法).

◇ 协商缓存

    ①ETag和If-None-Match要放在一起说: Etag是上一次加载资源时,服务器返回的response header,是对该资源的一种唯一标识,
    只要资源有变化,Etag就会重新生成.浏览器在下一次加载资源,向服务器发送请求时,会将上一次返回的Etag值放到request header
    里的If-None-Match里,服务器接收到If-None-Match的值后,会拿来跟该资源文件的Etag值做比较,如果相同,则表示资源文件没有
    发生变化,命中协商缓存.

    ②Last-Modified和If-Modified-Since要放在一起说: Last-Modified是该资源文件的最后一次修改时间,服务器会在
    response header里返回,同时浏览器会将这个值保存起来,在下一次发送请求时,放到request header里的
    If-Modified-Since里,服务器在接收到后做比较,如果相同则命中协商缓存.

    (图片)

◇ ETag和Last-Modified的作用类似,说一说他们的区别:

    ①首先在精度上,Etag要优于Last-Modified.Last-Modified的单位是秒,如果某个文件在一秒内被修改多次,那么它的
    Last-Modified其实并没有体现出来文件被修改,但是Etag每次都会改变,因此确保了精度;如果是负载均衡的服务器,各个服务器生成
    的Last-Modified也可能不一致.

    ②第二在性能上,Etag要逊于Last-Modified,毕竟Last-Modified只需要记录时间,而Etag需要服务器通过算法来计算出一个hash
    值.

    ③第三在优先级上,服务器校验优先考虑Etag.

● 浏览器缓存过程

浏览器第一次加载资源,服务器返回200,浏览器将资源文件从服务器上请求下载下来,并把response header及该请求的返回时间一并缓存;

下一次加载资源时,先比较当前时间和上一次返回200时的时间差,如果没有超过Cache-Control设置的max-age,则没有过期,命中强缓存,
不发请求直接从本地缓存读取该文件(如果浏览器不支持http1.1,则用Expires判断是否过期);如果已经过期,则向服务器发送header带有
If-None-Match和If-Modified-Since的请求;

服务器收到请求后,优先根据Etag的值判断被请求的文件有没有被做修改,Etag值一致则没有修改,命中协商缓存,返回304;不一致则有改动,
直接返回新的资源文件,带上新的Etag值并返回200;

如果服务器收到的请求没有Etag值,则将If-Modified-Since值和被请求的文件的最后修改时间做比对,一致则命中协商缓存,返回304;不
一致则返回新的资源文件,带上新的Last-Modified值并返回200;

◇ 用户行为对浏览器缓存的控制:

    地址栏访问,页面链接跳转是正常的用户行为,将会触发浏览器缓存机制;

    F5刷新,浏览器会设置max-age=0,跳过强缓存判断,会进行协商缓存判断;

    CTRL+F5刷新,跳过强缓存和协商缓存,直接从服务器拉取资源.





