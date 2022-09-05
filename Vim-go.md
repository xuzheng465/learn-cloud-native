https://github.com/fatih/vim-go/wiki/Tutorial



- Use `ctrl-]` or `gd` to jump to a definition, locally or globally
- Use `ctrl-t` to jump back to the previous location



```
Plug 'ctrlpvim/ctrlp.vim'
```

有了这个插件就可以使用这两个命令

```sh
:GoDecls
:GoDeclsDir
```

```sh
]] -> jump to next function
[[ -> jump to previous function
```

e typing `v]]` and then hit `]]` again to select the next function, until you've done with your selection.



we can quickly jump between a Go source code and its test file.

```
:GoAlternate
```



















## Resources

https://github.com/buoto/gotests-vim

