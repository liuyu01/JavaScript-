##### F5和Ctrl+F5的区别

一个是刷新，一个是强制刷新

按F5有时候一些内容是不会被更新的，而Ctrl+F5则所有内容都会被更新

具体区别是：

F5通常只是刷新本地缓存；

Ctrl+F5可以把INTERNET临时文件夹的文件删除再重新从服务器下载，也就是彻底刷新页面了

##### 浏览器缓存如何控制？

###### 方案一：无缓存

**说明**：浏览器向服务器请求资源m.png, 然后服务器响应请求--找到对应的m.png后发送给浏览器。 之后，浏览器再次向服务器请求m.png， 服务器又发回了同样的一张图片....循环往复.....

**优点**：浏览器请求，服务器响应，思路清楚简单容易实现。　　　

**缺点**：每次都请求同样的资源时，服务器也在不断地响应，这是非常浪费带宽的。

###### 方案二：有缓存-无更新

**说明：**　同样，浏览器向服务器请求资源ｍ.png,然后服务器找到后发送给浏览器，这时浏览器把m.png保存在本地（缓存）**,** 这样以后每次再请求m.png时就不需要向服务器要了，直接从本地取就行了，但是下次这个m.png的内容换了呢？

**优点：** 节省带宽（显然的，后续直接从本地取资源即可）。      

**缺点：** 如果服务器上的m.png内容改变，我就不能得到改变后的资源了，而是始终拿到本地的资源。

###### 方案三：有缓存-有更新

 **说明：** 浏览器向服务器请求资源m.png，然后服务器找到后发送给服务器，同时还附带额外信息---过期时间，如Expires: Friday,26 Feb 2017 10:11:22GMT。  然后浏览器将图片和过期时间同时保存在本地。 

 浏览器第二次向服务器请求资源，这时它会先查看过期事件是否已经达到，如果在过期事件之内，就直接使用本地缓存（**状态304**）； 如果超出了这个过期时间，就重新向服务器发送请求，服务器再次发回最新资源和最新的过期时间， 浏览器再次保存...

**优点：**  一方面可以在过期时间之内就可以不再重新请求资源，节省了带宽；另一方面也不会像第二种方案一样，而是可以得到新资源。

 **缺点：** 在过期时间之后就要重新请求资源，但是如果资源内容没改变呢？ 这次拿回的资源不就浪费了带宽吗？除此之外，这种时间格式复杂，容易比对出错。

###### 方案四：有缓存-有更新-更新机制加强

**说明：** 刚才的更新机制是发送来过期时间，而现在服务器在发送资源给浏览器的时候不再发送具体的时间，而是发送一个Cache-Control,这里可以包含各种信息。如Cache-Control: max-age=300; 这种方式和上面方案类似， 只是时间过期使用数字，不容易出错。另外Cache-Control还可以是下面的一些值：  

- Public---表示服务器发送的资源可以被任何中间节点缓存，如 Server -> proxy1 -> proxy2 -> Browser，proxy1 和 proxy2也可以缓存资源，这样，下次请求时proxy2就可以返回资源。
- Private---表示服务器发送的资源不可以被任何中间节点缓存。**
  **
- no-cache---表示不使用Cache-Control的缓存控制方式（强缓存），而是使用Etag 或者 Last-Modified（协商缓存）字段来控制缓存。
- no-store---表示真正地不用缓存方式（每次都请求最新的资源），Etag和Last-Modified也不用。
- max-age---表示当前资源的有效时间（就是强制缓存，用于替代HTTP1.0的Expires的方案）。

　**优点 ：** 使用max-age更加容易比对，其他的几个值使得缓存机制更加强大。 

​     **缺点：**同方案三，有可能导致浪费带宽。

###### 方案五：有缓存-有更新-更新机制完美

**说明：**为了解决方案四在300s后请求资源时得到了并未更新的资源而导致浪费带宽的情况，我们在给浏览器返回m.png图片时，不仅需要附加 Cache-Control: max-age=300; 再发送一个ETag字段,如 Etag:W/"e-dafdajio54fdaadf/q5w"。然后浏览器将图片和两个附加信息都保存起来， 300s内请求资源时，就从本地取，300s后请求资源时就将之前拿到的ETag信息随着请求发出，服务器接受到请求后先比对得到的ETag和服务器处图片当前的ETag，如果相同，则表示图片内容没变，就发送消息（不包含图片，**304**）；如果ETag改变，就发送新的m.png并且再发送一个新的Etag给浏览器保存，那么这时的max-age应该也是同样需要更新的，如此循环往复......

与Etag功能类似的是Last-Modified/if-Modified-Since ，当资源过期时（max-age超时），发现资源具有Last-Modified声明（是浏览器接收到的资源最新被修改的时间），于是发送请求时带上If-Modified-Since（即刚才的Last-Modified的时间）,web服务器收到请求时，将If-Modeified-Since时间的资源与当前资源对比， 如果没变， 就响应**HTTP304**，让浏览器使用缓存， 如果不是，就发送新的资源。 

 
