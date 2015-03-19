title: "vim rst table plugin"
date: 2015-03-12 14:50:21
tags:
---

### 安装vim-bridge

```
sudo pip install vim_bridge
```

### 安装vim rst table plugin

利用 bundle安装：
```
Bundle 'fire3/vim_rst_tables.vim'
```

然后在vim中运行命令

```
:BundleInstall
```

### 配置rst表格插件

`.vimrc`中增加:

```
" Add mappings, unless the user didn't want this.
" The default mapping is registered, unless the user remapped it already.
if !exists("no_plugin_maps") && !exists("no_rst_table_maps")
    if !hasmapto('ReformatTable(')
        noremap <silent> ,,c :call ReformatTable()<CR>
     endif
endif
```

### 使用范例

```
中文  测试
中英文abc  测试ab
```
在上面任意位置按下 `,,c` 即可出现:

```

+------------+---------+
| 中文       | 测试    |
+============+=========+
| 中英文abc  | 测试ab  |
+------------+---------+

```
