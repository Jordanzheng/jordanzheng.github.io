# Multiple commands for Linux shell

今天在写shell 脚本时，需要几条命令串起来执行，并且前面命令执行失败了，后面命令就不行了。上网google一下，找到解决办法，采用逻辑与 ** && ** 即可。

下面总结一下shell执行多命令的方法。

####  一、分号；分割

    
    
    command1;command2;command3;...

前面命令失败了不影响后面的命令执行。

####  二、逻辑与 &amp;&amp;

** 命令执行返回值为0表明执行成功 **
    
    
    command1 && command2 && command3 && ...

命令都用&amp;&amp;串接，从左到右执行，当前面的命令执行“成功”后才继续执行后面的命令。  
另外，在script文件中，如果某一行太长写不完，command1 太长，可以在行末，放置接续上行的符号”\”。

    
    
    command1 && \
    command2 && \
    command3 && ...

####  三、逻辑或 ||

    
    
    command1 || command2 || command3 || ...

从左到右执行，当前面的命令执行“失败”后才继续执行后面的命令。若前一个命令执行成功，就不会执行下一条了。


