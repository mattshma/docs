# vim配置

目前vim使用Vundle。

## delete键不能使用？
在~/.vimrc中设置`set backspace=2`

## no modules called ycm_core
在`~/.vim/bundle/YouCompleteMe`中执行`sudo ./install.py --clang-complete --gocode-completer --tern-completer --racer-completer --enable-debug`。

## 查看错误信息
在visual模式下，运行`:YcmToggleLogs stderr`。
