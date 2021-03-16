fatal: unable to access '.git/': LibreSSL SSL_connect: SSL_ERROR_SYSCALL in connection to github....

同样的问题，原来是我配置不正确的代理设置，这是检查和删除它们的方法。
首先打开您的git配置文件。

vi ~/.gitconfig
并确定是否设置了[http]或[https]部分。

由于我在中国访问Github的速度较慢，我曾经为git设置代理，但是最近我更改了本地代理端口，但忘记了git设置。

如果您有不正确的代理设置并决定删除它，只需执行：

git config --global --unset http.proxy
git config --global --unset https.proxy
一切都会好起来的。。。。。