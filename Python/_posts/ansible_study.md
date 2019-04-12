错误1
```
[zhangcd@ajkoffice-7-201 ansible-2.7.8]$ make rpm
Traceback (most recent call last):
  File "packaging/release/versionhelper/version_helper.py", line 9, in <module>
    from packaging.version import Version, VERSION_PATTERN
ImportError: No module named packaging.version
Makefile:40: *** "version_helper failed"。 停止。
```
https://stackoverflow.com/questions/44272205/ansible-error-no-module-named-packaging-version


错误2
```
ERROR: rst2man from docutils command is not installed but is required to build ./docs/man/man1/ansible-config.1 ./docs/man/man1/ansible-console.1 ./docs/man/man1/ansible-doc.1 ./docs/man/man1/ansible-galaxy.1 ./docs/man/man1/ansible-inventory.1 ./docs/man/man1/ansible-playbook.1 ./docs/man/man1/ansible-pull.1 ./docs/man/man1/ansible-vault.1
make[1]: *** [docs/man/man1/ansible.1] 错误 1
rm docs/man/man1/ansible.1.rst
make[1]: Leaving directory `/data/zhangcd/software/ansible-2.7.8'
make: *** [docs] 错误 2

```

```
pip install docutils
```

错误3
```
/bin/sh: rpmbuild: command not found
```

```
sudo yum install rpm-build
```


错误4
```
在使用pyenv环境下，安装ansible-2.7.8后无法启动：
[zhangcd@ajkoffice-7-201 ansible-2.7.8]$ ansible
Traceback (most recent call last):
  File "/data/zhangcd/.pyenv/versions/2.7.8/bin/ansible", line 4, in <module>
    __import__('pkg_resources').run_script('ansible==2.7.8', 'ansible')
  File "/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/pkg_resources/__init__.py", line 739, in run_script
    self.require(requires)[0].run_script(script_name, ns)
  File "/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/pkg_resources/__init__.py", line 1492, in run_script
    raise ResolutionError("No script named %r" % script_name)
```
检查pyenv的代码，它是在pyenv脚本中将ansible重新指向一个通用的shell脚本放在~/.pyenv/shims/目录下：
~/.pyenv/shims/ansible
```bash
#!/usr/bin/env bash
set -e
[ -n "$PYENV_DEBUG" ] && set -x

program="${0##*/}"
if [[ "$program" = "python"* ]]; then
  for arg; do
    case "$arg" in
    -c* | -- ) break ;;
    */* )
      if [ -f "$arg" ]; then
        export PYENV_FILE_ARG="$arg"
        break
      fi
      ;;
    esac
  done
fi

export PYENV_ROOT="/data/zhangcd/.pyenv"
exec "/data/zhangcd/.pyenv/libexec/pyenv" exec "$program" "$@"
```

这个脚本会把ansible命令使用/data/zhangcd/.pyenv/libexec/pyenv包装执行

在这个脚本里，首先会将ansible的实际所在目录包含到PATH变量里，然后通过command命令来查找真实的ansible的路径来执行,
```bash
# 添加PATH变量搜索目录
for plugin_bin in "${PYENV_ROOT}/plugins/"*/bin; do
  PATH="${plugin_bin}:${PATH}"
done
export PATH="${bin_path}:${PATH}"

# 执行真实的ansible命令
command="$1"
case "$command" in
"" )
  { pyenv---version
    pyenv-help
  } | abort
  ;;
-v | --version )
  exec pyenv---version
  ;;
-h | --help )
  exec pyenv-help
  ;;
* )
  command_path="$(command -v "pyenv-$command" || true)"
  [ -n "$command_path" ] || abort "no such command \`$command'"

  shift 1
  if [ "$1" = --help ]; then
    if [[ "$command" == "sh-"* ]]; then
      echo "pyenv help \"$command\""
    else
      exec pyenv-help "$command"
    fi
  else
    #echo "test_zcd:${command_path}" 1>&2
    exec "$command_path" "$@"
  fi
  ;;
esac
```
知道pyenv的大概原理后，开始查找/data/zhangcd/.pyenv/versions/2.7.8/bin/ansible这个脚本的问题。

这个脚本里只有一行代码：
```python
__import__('pkg_resources').run_script('ansible==2.7.8', 'ansible')
```
就是使用pkg_resources包的run_script函数运行ansible脚本，这个文件也是相当于是一个包装脚本，没有实际完成功能
在对pkg_resources包的run_script函数进行调试时，发现run_script调用的脚本是这样一个脚本：
```
> /data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/pkg_resources/__init__.py(1565)_has()
-> def _has(self, path):
(Pdb) p path
'/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8.dist-info/scripts/ansible'
```
这个路径是通过ansible的egg_info和scripts目录以及ansible命令拼接而成的，这个地方self.egg_info为
```
/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8.dist-info/
```

通过查看实际目录，发现/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/这个目录下有三个ansible相关的目录：
```
/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible
/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8.dist-info/
/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8-py2.7.egg/
```
因为我在使用pip install ansible安装后找不到ansible命令，又用源码安装了一遍，所以这个地方有三个ansible的目录。
通过安装和卸载的测试发现，pip intall ansible安装的包是：
```
/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible
/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8.dist-info/
```
通过源码make install安装的包是：
```
/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8-py2.7.egg/
```

而名称为ansible-2.7.8-py2.7.egg的目录下是有ansible脚本的：
```
[zhangcd@ajkoffice-7-201 python2.7]$ tree /data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8-py2.7.egg/EGG-INFO/
/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8-py2.7.egg/EGG-INFO/
├── dependency_links.txt
├── not-zip-safe
├── PKG-INFO
├── requires.txt
├── scripts
│   ├── ansible
│   ├── ansible-config
│   ├── ansible-connection
│   ├── ansible-console
│   ├── ansible-doc
│   ├── ansible-galaxy
│   ├── ansible-inventory
│   ├── ansible-playbook
│   ├── ansible-pull
│   └── ansible-vault
├── SOURCES.txt
└── top_level.txt
```
为什么pkg_resources没有把/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8-py2.7.egg/EGG-INFO/作为ansible的self.egg_info值？

通过跟踪源码，发现ansible的self.egg_info变量是在~/.pyenv/versions/2.7.8/lib/python2.7/site-packages/pkg_resources/__init__.py的_initialize_master_working_set函数中被构造出来的：
```python
@_call_aside
def _initialize_master_working_set():
    """
    Prepare the master working set and make the ``require()``
    API available.

    This function has explicit effects on the global state
    of pkg_resources. It is intended to be invoked once at
    the initialization of this module.

    Invocation by other packages is unsupported and done
    at their own risk.
    """
    working_set = WorkingSet._build_master()
    _declare_state('object', working_set=working_set)
```
pkg_resources通过递归遍历sys.path里的所有目录来引入包，其中包括：
脚本所在目录、lib/python27.zip文件、lib/python2.7/site-packages目录
```
https://docs.python.org/3/library/sys.html#sys.path
https://stackoverflow.com/a/38403654/2873403
```

通过调试发现ansible的egg_info对应的目录是/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8.dist-info/
当遍历到/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8-py2.7.egg后，发现ansible已经被加载了，就忽略跳过了：
```python
def add(self, dist, entry=None, insert=True, replace=False):
    """Add `dist` to working set, associated with `entry`

    If `entry` is unspecified, it defaults to the ``.location`` of `dist`.
    On exit from this routine, `entry` is added to the end of the working
    set's ``.entries`` (if it wasn't already present).

    `dist` is only added to the working set if it's for a project that
    doesn't already have a distribution in the set, unless `replace=True`.
    If it's added, any callbacks registered with the ``subscribe()`` method
    will be called.
    """
    if insert:
        dist.insert_on(self.entries, entry, replace=replace)

    if entry is None:
        entry = dist.location
    keys = self.entry_keys.setdefault(entry, [])
    keys2 = self.entry_keys.setdefault(dist.location, [])
	# 当dist.location为/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8-py2.7.egg，key为ansible时，发现self.by_key已经有key为ansible的项了，直接跳过
    if not replace and dist.key in self.by_key:
        # ignore hidden distros
        return

    self.by_key[dist.key] = dist
    if dist.key not in keys:
        keys.append(dist.key)
    if dist.key not in keys2:
        keys2.append(dist.key)
    self._added_new(dist)
```

至此已经知道发生这种错误的大概原因了，pyenv在执行ansible脚本时，加载的ansible包加载了另一个重复的包，这个包的目录结构与预定的不一样。

那到底是怎么发生这个问题呢？
印象中是这样做的：
```
1. 通过pip install ansible安装ansible包后，发现ansible命令不能执行（其实是pyenv的问题，没有将PATH中的命令加载上），怀疑可能这个包没有可执行命令
2. 于是再下载ansible源码安装
3. 此时ansible的脚本命令就被替换为内容只有一行的包装执行脚本：
__import__('pkg_resources').run_script('ansible==2.7.8', 'ansible')
实际的执行脚本在/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8-py2.7.egg/EGG-INFO/目录下
4. python的import机制把/data/zhangcd/.pyenv/versions/2.7.8/lib/python2.7/site-packages/ansible-2.7.8.dist-info/作为ansible的包位置，运行ansible命令无法找到该脚本
```

解决方法也很简单：
```
1. 先卸载ansible:
	pip uninstall ansible
2. 然后使用源码方式或者pip安装方式其中一种，必须只能选择一种安装：
	pip install ansible 或者 make install
3. 然后执行pyenv rehash
	此时就可以执行ansible命令了
```

过程中了解到了pyenv的相关命令信息：
```
shopt命令：
	http://man.hubwiz.com/docset/Bash.docset/Contents/Resources/Documents/bash/The-Shopt-Builtin.html#The-Shopt-Builtin
	https://www.linuxtopia.org/online_books/advanced_bash_scripting_guide/globbingref.html
	https://unix.stackexchange.com/questions/32409/set-and-shopt-why-two
	https://bash.cyberciti.biz/guide/Setting_shell_options
	https://bash.cyberciti.biz/guide/Shopt
command命令：
	https://askubuntu.com/questions/512770/what-is-use-of-command-command
type命令：
	https://unix.stackexchange.com/questions/89048/how-to-test-if-command-is-alias-function-or-binary
```

还有python的pdb调试：
```
python的环境变量:
	https://docs.python.org/2/using/cmdline.html#environment-variables
pdb调试：
	https://docs.python.org/2/library/pdb.html
	https://www.cnblogs.com/wei-li/archive/2012/05/02/2479082.html
```
