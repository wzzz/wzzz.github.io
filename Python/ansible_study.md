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
