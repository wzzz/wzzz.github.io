# perl变量

使用my申明局部变量
my $a = 1;

perl的变量类型有三种：
标量，数组，哈希
标量变量在声明时用$做前缀标识
数组变量在声明时用@做前缀标识
哈希变量在声明时用%做前缀标识

perl中的引用类似于C的指针，它的类型本质上是标量，但是对于不同类型的变量的引用，它可以是标量引用、数据引用、哈希引用
http://www.runoob.com/perl/perl-references.html
http://www.cnblogs.com/softwaretesting/archive/2011/07/26/2117730.html


# perl文件操作
http://www.runoob.com/perl/perl-files.html

只读打开文件
```perl
#!/usr/bin/perl
 
open(DATA, "<file.txt") or die "file.txt 文件无法打开, $!";
 
while(<DATA>){
   print "$_";
}
```


# perl的二维数组
https://www.cnblogs.com/visayafan/archive/2011/10/29/2228596.html#sec-1
http://www.runoob.com/perl/perl-arrays.html
perl数组本质上是一个一维数组，它的每个元素是一个引用，然后这些引用又是指向一维数组，这个一维数组中的元素才是二维数组中真正元素。

## perl循环遍历二维数组
```perl
my @arr = ();
push @arr, [1, 2, 3];
push @arr, [11, 22, 33];
foreach $row (@arr) {
    my @arr = @$row;
    print "row cell count is @arr\n";
    foreach $cell (@arr) {
        print "\tcell $cell\n";
    }
}
```

# perl匿名数组和匿名哈希
https://www.cnblogs.com/f-ck-need-u/p/9718238.html
构造匿名对象时，其实是创建了一个匿名对象的引用，然后这个引用指向的对象是数组或者哈希
```perl
my @name=('fairy',['longshuai','wugui','xiaofang']); # 创建了一个数组，变量名为@name
my $name=('fairy',['longshuai','wugui','xiaofang']); # 创建了一个数组，把这个数组的引用赋值给标量$name

my @name=['fairy',['longshuai','wugui','xiaofang']]; # 创建了一个匿名数组和一个名为@name的数组，把这个匿名数组的引用赋值@name的第一个元素
my $name=['fairy',['longshuai','wugui','xiaofang']]; # 创建了一个匿名数组，把这个数组的引用赋值给标量$name
```

# perl的字符串比较

字符串比较用 eq 操作符
```perl
if($name eq "hello")
{
	print "$name\n";
}
```

# perl的正则表达式匹配
```perl
if($name =~ /:/)
{
	print $name."中包含:\n";
}
```
