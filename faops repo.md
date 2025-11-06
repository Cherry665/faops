1111
# 01-help.bats  

## bats代码①:  

```bats
@test "run faops" {  
    run $BATS_TEST_DIRNAME/../faops help  
    assert_success  
}  
```

## 可以运行的bash代码①:

检测 faops 程序是否成功安装并能够成功执行，成功输出faops具体信息和“ Success ”，失败输出“ Failed ”

```bash
faops help && echo "Success" || echo "Failed"
```

## bats代码②:  

```bats
@test "help: contents" {
    run $BATS_TEST_DIRNAME/../faops help
    echo "${output}" | grep "Usage"
    assert_success
}
```

## 可以运行的bash代码②:  

检查 faops help 的输出结果中是否包含 Usage ，包含输出“ faops help output contains 'Usage'”，不包含输出“ faops help output doesn't contains 'Usage'”  

```bash
faops help | grep "Usage"
if [ $? -eq 0 ]; then  
   echo "faops help output contains 'Usage' "
else
   echo "faops help output doesn't contains 'Usage' "
fi  
```

## bats代码③:  

```bats
@test "help: lines of contents" {
    run $BATS_TEST_DIRNAME/../faops help
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal "27" "${output}"
}
```

## 可以运行的bash代码③:  

计算 faops help 输出的行数，并判断行数是否等于 27  

```bash
line_count=$(echo "$(faops help)" | wc -l | xargs echo)
echo "line_count:$line_count"
if [ $line_count -eq 27 ] ;then
   echo "The line number of faops help is 27"
else
   echo "The line number of faops help isn't 27,the actual line number of faops help is $line_count"
fi
```

# 02-count.bats  
## bats代码①:  
```
@test "count: read from file" {
    run bash -c "$BATS_TEST_DIRNAME/../faops count $BATS_TEST_DIRNAME/ufasta.fa | head -n 2"
    assert_equal "#seq${tab}len${tab}A${tab}C${tab}G${tab}T${tab}N" "${lines[0]}"
    assert_equal "read0${tab}359${tab}99${tab}89${tab}92${tab}79${tab}0" "${lines[1]}"
}  
```
## 可以运行的bash代码①:  
计算 ufasta.fa 文件中序列的序列名、序列长度和各碱基（A/C/G/T/N）的数量，只取前两行输出（去掉 head 可以输出全部）
```
cd $HOME/faops/test
faops count ufasta.fa | head -n 2
#或 faops count $HOME/faops/test/ufasta.fa | head -n 2
```
## bats代码②: 
```
@test "count: read from gzipped file" {
    run bash -c "$BATS_TEST_DIRNAME/../faops count $BATS_TEST_DIRNAME/ufasta.fa.gz | head -n 2"
    assert_equal "#seq${tab}len${tab}A${tab}C${tab}G${tab}T${tab}N" "${lines[0]}"
    assert_equal "read0${tab}359${tab}99${tab}89${tab}92${tab}79${tab}0" "${lines[1]}"
}
```
## 可以运行的bash代码②:
计算 ufasta.fa.gz 压缩文件中序列的序列名、序列长度和各碱基（A/C/G/T/N）的数量，与 ① 类似
```
cd $HOME/faops/test
faops count ufasta.fa.gz | head -n 2
#或 faops count $HOME/faops/test/ufasta.fa.gz | head -n 2
```
## bats代码③:
```
@test "count: read from stdin" {
    run bash -c "cat $BATS_TEST_DIRNAME/ufasta.fa | $BATS_TEST_DIRNAME/../faops count stdin | head -n 2"
    assert_equal "#seq${tab}len${tab}A${tab}C${tab}G${tab}T${tab}N" "${lines[0]}"
    assert_equal "read0${tab}359${tab}99${tab}89${tab}92${tab}79${tab}0" "${lines[1]}"
}
```
## 可以运行的bash代码③:
先从 ufasta.fa 文件中读取内容传输给 stdin（标准输入），后从 stdin 中读取内容，计算序列的序列名、序列长度和各碱基（A/C/G/T/N）的数量，只取前两行输出。比前面更灵活，可以处理动态生成的数据
```
cd $HOME/faops/test
cat ufasta.fa | faops count stdin | head -n 2
```
## bats代码④:
```
@test "count: lines of result" {
    run $BATS_TEST_DIRNAME/../faops count $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal 52 "${output}"
}
```
## 可以运行的bash代码④:
计算 faops count ufasta.fa 输出结果的行数  
• 50条序列 + 1个表头 + total = 52行
```
cd $HOME/faops/test
line_count=$(echo "$(faops count ufasta.fa)" | wc -l | xargs echo)
echo "line_count:$line_count"
```
## bats代码⑤:
```
@test "count: mixture of stdin and actual file" {
    run bash -c "cat $BATS_TEST_DIRNAME/ufasta.fa | $BATS_TEST_DIRNAME/../faops count stdin $BATS_TEST_DIRNAME/ufasta.fa"
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal 102 "${output}"
}
```
## 可以运行的bash代码⑤:
利用 faops count 同时处理 stdin（标准输入）和原文件，计算两者行数的总和  
• 从stdin读取：50条序列  
• 从原文件读取：50条序列  
• 总行数：50 + 50 + 1个表头 + total = 102行  
```
cd $HOME/faops/test
line_count=$(echo "$(cat ufasta.fa | faops count stdin ufasta.fa)" | wc -l | xargs echo)
echo "line_count:$line_count"
```
## bats代码⑥:
```
@test "count: sum of sizes" {
    run bash -c "
        $BATS_TEST_DIRNAME/../faops count $BATS_TEST_DIRNAME/ufasta.fa \
            | perl -ne '/^total\t(\d+)/ and print \$1'
    "
    assert_equal 9317 "${output}"
}
```
## 可以运行的bash代码⑥:
计算 ufasta.fa 文件中所有序列长度的总和（count）  
以下为perl语言解释：  
• ^：匹配行首  
• total：匹配文字"total"  
• \t：匹配制表符  
• (\d+)：匹配一个或多个数字，并用括号捕获（保存到$1）  
• print $1：打印正则表达式中第一个括号捕获的内容（即数字部分）
```
cd $HOME/faops/test
total=$(faops count ufasta.fa | perl -ne '/^total\t(\d+)/ and print $1')
echo "total:$total"
```
## bats代码⑦:
```
@test "faCount: without cpg" {
    if ! hash faCount 2>/dev/null ; then
        skip "Can't find faCount"
    fi

    exp=$(faCount $BATS_TEST_DIRNAME/ufasta.fa | perl -anle 'print join qq{\t}, @F[0 .. 6]')
    res=$($BATS_TEST_DIRNAME/../faops count $BATS_TEST_DIRNAME/ufasta.fa)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑦:
对比 faops count 和 facount 的输出结果是否一致  
以下为perl语言解释：  
• -a：自动分割模式，将每行按空白分割到 @F 数组  
• -n：逐行处理输入  
• -l：自动处理换行符  
• @F[0 .. 6]：取数组的前7个元素（第1-7列）  
• join qq{\t}：用制表符连接这些元素  
```
if ! hash faCount 2>/dev/null ; then        
   echo "Can't find faCount"
else
exp=$(faCount ufasta.fa | perl -anle 'print join "\t", @F[0..6]')
res=$(faops count ufasta.fa)
   if [ "$exp" = "$res" ]; then
       echo "The outputs of faops_count and faCount are the same"
   else
       echo "The outputs of faops_count and faCount are different"
   fi
fi
```
# 03-size.bats  
## bats代码①:
```
@test "size: read from file" {
    run bash -c "$BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa | head -n 2"
    assert_equal "read0${tab}359" "${lines[0]}"
    assert_equal "read1${tab}106" "${lines[1]}"
}
```
## 可以运行的bash代码①:
计算 ufasta.fa 文件中序列的长度（只输出前两条）  
```
cd $HOME/faops/test
faops size ufasta.fa | head -n 2
```
## bats代码②:
```
@test "size: read from gzipped file" {
    run bash -c "$BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa.gz | head -n 2"
    assert_equal "read0${tab}359" "${lines[0]}"
    assert_equal "read1${tab}106" "${lines[1]}"
}
```
## 可以运行的bash代码②:
计算 ufasta.fa.gz 压缩文件中序列的长度，与①类似  
```
cd $HOME/faops/test
faops size ufasta.fa.gz | head -n 2
```
## bats代码③:
```
@test "size: read from stdin" {
    run bash -c "cat $BATS_TEST_DIRNAME/ufasta.fa | $BATS_TEST_DIRNAME/../faops size stdin | head -n 2"
    assert_equal "read0${tab}359" "${lines[0]}"
    assert_equal "read1${tab}106" "${lines[1]}"
}
```
## 可以运行的bash代码③:
先从 ufasta.fa 文件中读取内容传输给 stdin（标准输入），后从 stdin 中读取内容，计算序列的长度  
```
cd $HOME/faops/test
cat ufasta.fa | faops size stdin | head -n 2
```
## bats代码④:
```
@test "size: lines of result" {
    run $BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal 50 "${output}"
}
```
## 可以运行的bash代码④:
计算 faops size ufasta.fa 输出结果的行数，与 count 不同的是没有表头和total  
```
cd $HOME/faops/test
line_count=$(echo "$(faops size ufasta.fa)" | wc -l | xargs echo)
echo "line_count:$line_count"
```
## bats代码⑤:
```
@test "size: mixture of stdin and actual file" {
    run bash -c "cat $BATS_TEST_DIRNAME/ufasta.fa | $BATS_TEST_DIRNAME/../faops size stdin $BATS_TEST_DIRNAME/ufasta.fa"
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal 100 "${output}"
}
```
## 可以运行的bash代码⑤:
利用 faops size 同时处理 stdin（标准输入）和原文件，计算两者行数的总和  
```
cd $HOME/faops/test
line_count=$(echo "$(cat ufasta.fa | faops size stdin ufasta.fa)" | wc -l | xargs echo)
echo "line_count:$line_count"
```
## bats代码⑥:
```
@test "size: sum of sizes" {
    run bash -c "
        $BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa \
            | perl -ane '\$c += \$F[1]; END { print qq{\$c\n} }'
    "
    assert_equal 9317 "${output}"
}
```
## 可以运行的bash代码⑥:
计算 ufasta.fa 文件中所有序列长度的总和（size）  
• -a：自动分割模式，将每行按空白分割到 @F 数组  
• -n：逐行处理输入  
• -e：执行后面的 Perl 代码  
• \$c += \$F[1]：累加第二列（序列长度）的值  
• END { print qq{\$c\n} }：处理完所有行后打印累加结果  
```
cd $HOME/faops/test
total=$(faops size ufasta.fa | perl -ane '$c += $F[1]; END { print qq{$c\n} }')
echo "total:$total"
```
# 04-frag.bats
## bats代码①:
```
@test "frag: from first sequence" {
    res=$($BATS_TEST_DIRNAME/../faops frag $BATS_TEST_DIRNAME/ufasta.fa 1 10 stdout | grep -v "^>")
    assert_equal "tCGTTTAACC" "${res}"
}
```
## 可以运行的bash代码①:
`frag`可以提取 ufasta.fa 文件中的序列片段，提取第 1-10 个碱基。`grep -v "^>"`可以删除以“>”开头的行，即描述行  
```
cd $HOME/faops/test
res=$(faops frag ufasta.fa 1 10 stdout | grep -v "^>")
echo "The extracted sequence is:$res"
```
## bats代码②:
```
@test "frag: from specified sequence" {
    res=$($BATS_TEST_DIRNAME/../faops some $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout \
        | $BATS_TEST_DIRNAME/../faops frag stdin 1 10 stdout \
        | grep -v "^>")
    assert_equal "AGCgCcccaa" "${res}"
}
```
## 可以运行的bash代码②:
`some`可以提取指定序列，`<(echo read12)`提供序列名为 read12 的序列，输出到 stdout  
`frag`可以从 stdin 中提取第 1-10 个碱基，输出到 stdout  
```
cd $HOME/faops/test
res=$(faops some ufasta.fa <(echo read12) stdout | faops frag stdin 1 10 stdout | grep -v "^>")
echo "The extracted sequence is:$res"
```
# 05-rc.bats
## bats代码①:
```
@test "rc: output same length" {
    exp=$($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa)
    res=$($BATS_TEST_DIRNAME/../faops rc -n $BATS_TEST_DIRNAME/ufasta.fa stdout \
        | $BATS_TEST_DIRNAME/../faops size stdin)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码①:
可以验证利用`faops rc -n`得到的反向互补序列与原始序列长度相等
```
cd $HOME/faops/test
exp=$(faops size ufasta.fa)
res=$(faops rc -n ufasta.fa stdout | faops size stdin)
#echo "The length of the original sequence:$exp"
#echo "The length of the reverse complementary sequence:$res"
if [ "$exp" = "$res" ]; then
   echo "The length of the reverse complementary sequence is equal to the original sequence"
else
   echo "The length of the reverse complementary sequence is unequal to the original sequence"
fi
```
## bats代码②:
```
@test "rc: double rc" {
    exp=$($BATS_TEST_DIRNAME/../faops filter $BATS_TEST_DIRNAME/ufasta.fa stdout)
    res=$($BATS_TEST_DIRNAME/../faops rc -n $BATS_TEST_DIRNAME/ufasta.fa stdout \
        | $BATS_TEST_DIRNAME/../faops rc -n stdin stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码②:
验证双重反向互补操作会恢复原始序列（fa文件）  
`faops filter`用来清理数据格式？  
```
cd $HOME/faops/test
exp=$(faops filter ufasta.fa stdout)    
res=$(faops rc -n ufasta.fa stdout | faops rc -n stdin stdout)
if [ "$exp" = "$res" ]; then   
   echo "The length of the double reverse complementary sequence is equal to the original sequence"
else   
   echo "The length of the double reverse complementary sequence is unequal to the original sequence"
fi
```
## bats代码③:
```
@test "rc: double rc (gz)" {
    exp=$($BATS_TEST_DIRNAME/../faops filter $BATS_TEST_DIRNAME/ufasta.fa stdout)
    res=$($BATS_TEST_DIRNAME/../faops rc -n $BATS_TEST_DIRNAME/ufasta.fa.gz stdout \
        | $BATS_TEST_DIRNAME/../faops rc -n stdin stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码③:
验证双重反向互补操作会恢复原始序列（fa.gz文件）  
```
cd $HOME/faops/test
exp=$(faops filter ufasta.fa stdout)    
res=$(faops rc -n ufasta.fa.gz stdout | faops rc -n stdin stdout)
if [ "$exp" = "$res" ]; then      
   echo "The length of the double reverse complementary sequence is equal to the original sequence"
else      
   echo "The length of the double reverse complementary sequence is unequal to the original sequence"
fi
```
## bats代码④:
```
@test "rc: perl regex" {
    paste <($BATS_TEST_DIRNAME/../faops rc -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -v '^>') \
        <($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -v '^>') \
        | perl -ane '
            $F[0] = uc($F[0]);
            $F[1] =~ tr/ACGTacgt/TGCATGCA/;
            $F[1] = reverse($F[1]);
            exit(1) unless $F[0] eq $F[1]
        '
    assert_success
}
```
## 可以运行的bash代码④:
利用perl验证`rc -l`反向互补的正确性  
• `rc -l 0`可以生成反向互补序列，且无长度限制  
• `paste`可以将两个命令的输出按列合并，每列包含反向互补序列 + 制表符 + 原始序列  
• `$F[0] = uc($F[0])`： 将反向互补序列转为大写  
• `$F[1] =~ tr/ACGTacgt/TGCATGCA/`： 对原始序列进行碱基互补  
• `$F[1] = reverse($F[1])`: 反转序列  
• `$F[0] ne $F[1]`: ne 为perl中的不等运算符  
• `$.`表示当前行号  
• 退出状态码1表示失败，0表示成功，`eof`判断是否到达文件末尾  
```
cd $HOME/faops/test
paste \
  <(faops rc -l 0 ufasta.fa stdout | grep -v '^>') \
  <(faops filter -l 0 ufasta.fa stdout | grep -v '^>') \
| perl -ane '
    $F[0] = uc($F[0]);
    $F[1] =~ tr/ACGTacgt/TGCATGCA/;
    $F[1] = reverse($F[1]);
    if ($F[0] ne $F[1]) {
        print "Mismatch at line $.: $F[0] vs $F[1]\n";
        exit(1);
    }
    exit(0) if eof;
'
if [ $? -eq 0 ]; then
    echo "Perl reverse complement algorithm verification successful"
else
    echo "Perl reverse complement algorithm verification failed"
fi
```
## bats代码⑤:
```
@test "rc: with list.file" {
    exp=">RC_read47"
    res=$($BATS_TEST_DIRNAME/../faops rc -l 0 -f <(echo read47) $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>RC_')

    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑤:
利用列表文件对指定序列进行反向互补操作  
• `-f <(echo read47)`只处理序列“read47”
```
cd $HOME/faops/test
exp=">RC_read47"    
res=$(faops rc -l 0 -f <(echo read47) ufasta.fa stdout | grep '^>RC_')
if [ "$exp" = "$res" ];then
   echo "The reverse complementary sequence was successfully generated using the list file"
else
   echo "Failed,exp:$exp;res:$res"
fi
```
# 06-one.bats
## bats代码①:
```
@test "one: inline names" {
    exp=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -A 1 '^>read12')
    res=$($BATS_TEST_DIRNAME/../faops one -l 0 $BATS_TEST_DIRNAME/ufasta.fa read12 stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码①:
利用`faops one`提取单个指定序列  
• `grep -A 1`可以显示匹配行及其后面一行内容
```
cd $HOME/faops/test
exp=$(faops filter -l 0 ufasta.fa stdout | grep -A 1 '^>read12')    
res=$(faops one -l 0 ufasta.fa read12 stdout)
if [ "$exp" = "$res" ];then   
   echo "faops one correctly extracts individual sequences"
else   
   echo "Failed,exp:$exp;res:$res"
fi
```
## bats代码②:
```
@test "faSomeRecords: inline names" {
    if ! hash faSomeRecords 2>/dev/null ; then
        skip "Can't find faSomeRecords"
    fi

    exp=$(faSomeRecords $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops one $BATS_TEST_DIRNAME/ufasta.fa read12 stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码②:
借助faSomeRecords验证`faops one`是否正确提取单个指定序列，与①类似
```
cd $HOME/faops/test
if ! hash faSomeRecords 2>/dev/null ; then           
   echo "Can't find faSomeRecords"
else
exp=$(faSomeRecords ufasta.fa <(echo read12) stdout | grep '^>')
res=$(faops one ufasta.fa read12 stdout | grep '^>')   
   if [ "$exp" = "$res" ]; then       
      echo "The outputs of faops_one and faSomeRecords are the same"   
   else       
      echo "The outputs of faops_one and faSomeRecords are different"   
   fi
fi
```
# 07-some.bats
## bats代码①:
```
@test "some: inline names" {
    exp=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -A 1 '^>read12')
    res=$($BATS_TEST_DIRNAME/../faops some -l 0 $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码①:
`faops some`可以用来提取多条指定序列  
将`<(echo read12)`替换为`<(echo -e "read12\nread13")`可以提取多条序列  
```
cd $HOME/faops/test
exp=$(faops filter -l 0 ufasta.fa stdout | grep -A 1 '^>read12')    
res=$(faops some -l 0 ufasta.fa <(echo read12) stdout)
if [ "$exp" = "$res" ]; then             
   echo "Faops some correctly extracts sequences"      
else             
   echo "Failed,exp：$exp;res:$res"      
fi
```
## bats代码②:
```
@test "faSomeRecords: inline names" {
    if ! hash faSomeRecords 2>/dev/null ; then
        skip "Can't find faSomeRecords"
    fi

    exp=$(faSomeRecords $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops some $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码②:
借助faSomeRecords验证`faops some`是否正确提取指定序列，与①类似  
```
cd $HOME/faops/test
if ! hash faSomeRecords 2>/dev/null ; then              
   echo "Can't find faSomeRecords"
else
exp=$(faSomeRecords ufasta.fa <(echo read12) stdout | grep '^>')
res=$(faops some ufasta.fa <(echo read12) stdout | grep '^>')    
   if [ "$exp" = "$res" ]; then             
      echo "The outputs of faops_some and faSomeRecords are the same"      
   else             
      echo "The outputs of faops_some and faSomeRecords are different"      
   fi
fi
```
## bats代码③:
```
@test "faSomeRecords: exclude" {
    if ! hash faSomeRecords 2>/dev/null ; then
        skip "Can't find faSomeRecords"
    fi

    exp=$(faSomeRecords -exclude $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops some -i $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码③:
利用`faops some -i`可以排除指定序列，保留剩余序列
```
cd $HOME/faops/test
if ! hash faSomeRecords 2>/dev/null ; then              
   echo "Can't find faSomeRecords"
else
exp=$(faSomeRecords -exclude ufasta.fa <(echo read12) stdout | grep '^>')
res=$(faops some -i ufasta.fa <(echo read12) stdout | grep '^>')    
   if [ "$exp" = "$res" ]; then             
      echo "The outputs of faops_some and faSomeRecords are the same"      
   else             
      echo "The outputs of faops_some and faSomeRecords are different"      
   fi
fi
```
# 08-filter.bats
## bats代码①:
```
@test "filter: as formatter, sequence in one line" {
    exp=$($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa | wc -l | xargs echo)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | wc -l | xargs echo)
    assert_equal "$(( exp * 2 ))" "$res"
}
```
## 可以运行的bash代码①:
`faops filter -l 0`可以格式化序列，且每条序列在一行上  
每个格式化后的序列占两行：1行头信息 + 1行序列数据  
```
cd $HOME/faops/test
exp=$(faops size ufasta.fa | wc -l | xargs echo)    
res=$(faops filter -l 0 ufasta.fa stdout | wc -l | xargs echo)
echo "exp:$exp"
echo "res:$res"
```
## bats代码②:
```
@test "filter: as formatter, blocked fasta files" {
    exp=$($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa | wc -l | xargs echo)
    res=$($BATS_TEST_DIRNAME/../faops filter -b $BATS_TEST_DIRNAME/ufasta.fa stdout | wc -l | xargs echo)
    assert_equal "$(( exp * 3 ))" "$res"
}
```
## 可以运行的bash代码②:
`faops filter -b`可以生成分块格式化的fa文件，后每条序列占3行，序列后面有一行空行  
```
cd $HOME/faops/test
exp=$(faops size ufasta.fa | wc -l | xargs echo)    
res=$(faops filter -b ufasta.fa stdout | wc -l | xargs echo)
echo "exp:$exp"
echo "res:$res"
```
## bats代码③:
```
@test "filter: as formatter, identical headers" {
    exp=$(grep '^>' $BATS_TEST_DIRNAME/ufasta.fa)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码③:
使用`faops filter`格式化序列后描述行信息不变  
```
cd $HOME/faops/test
exp=$(grep '^>' ufasta.fa)    
res=$(faops filter -l 0 ufasta.fa stdout | grep '^>')
if [ "$exp" = "$res" ]; then                   
   echo "The description line information remains unchanged after formatting the sequence"         
else                   
   echo "The description line information changes after formatting the sequence"         
fi
```
## bats代码④:
```
@test "filter: as formatter, identical sequences" {
    exp=$(grep -v '^>' $BATS_TEST_DIRNAME/ufasta.fa | perl -ne 'chomp; print')
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -v '^>' | perl -ne 'chomp; print')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码④:
使用`faops filter`格式化序列后序列信息不变  
• `chomp`：移除每行的换行符  
• `print`：连续打印（不移除序列字符间的换行）  
• 可以去除描述行，将所有序列数据连接成一个长字符串  
```
cd $HOME/faops/test
exp=$(grep -v '^>' ufasta.fa | perl -ne 'chomp; print')    
res=$(faops filter -l 0 ufasta.fa stdout | grep -v '^>' | perl -ne 'chomp; print')
if [ "$exp" = "$res" ]; then                      
   echo "The sequence information remains unchanged after formatting the sequence"         
else                      
   echo "The sequence information changes after formatting the sequence"         
fi
```
## bats代码⑤:
```
@test "filter: as formatter, identical sequences (gz)" {
    exp=$(grep -v '^>' $BATS_TEST_DIRNAME/ufasta.fa | perl -ne 'chomp; print')
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa.gz stdout | grep -v '^>' | perl -ne 'chomp; print')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑤:
使用`faops filter`格式化 fa.gz 压缩文件的序列后序列信息不变  
```
cd $HOME/faops/test
exp=$(grep -v '^>' ufasta.fa | perl -ne 'chomp; print')    
res=$(faops filter -l 0 ufasta.fa.gz stdout | grep -v '^>' | perl -ne 'chomp; print')
if [ "$exp" = "$res" ]; then                      
   echo "The sequence information remains unchanged after formatting the sequence"         
else                      
   echo "The sequence information changes after formatting the sequence"         
fi
```
## bats代码⑥:
```
@test "filter: as formatter, identical sequences (gz) with -N" {
    exp=$(grep -v '^>' $BATS_TEST_DIRNAME/ufasta.fa | perl -ne 'chomp; print')
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 -N $BATS_TEST_DIRNAME/ufasta.fa.gz stdout | grep -v '^>' | perl -ne 'chomp; print')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑥:
`faops filter -l 0 -N`中的`-N`参数可以将所有IUPAC模糊碱基代码（如M、R、S、W、Y、K等）转换为标准的N，保持A、G、C、T不变  
```
cd $HOME/faops/test
exp=$(grep -v '^>' ufasta.fa | perl -ne 'chomp; print')    
res=$(faops filter -l 0 -N ufasta.fa.gz stdout | grep -v '^>' | perl -ne 'chomp; print')
if [ "$exp" = "$res" ]; then                      
   echo "The sequence information remains unchanged after formatting the sequence"         
else                      
   echo "The sequence information changes after formatting the sequence"         
fi
```
## bats代码⑦:
```
@test "filter: convert IUPAC to N" {
    exp=$(printf ">read\n%s\n" ANNG)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 -N <(printf ">read\n%s\n" AMRG) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑦:
利用`faops filter -l 0 -N`的`-N`参数将所有IUPAC模糊碱基代码（M、R）转换为标准的N  
M=A/C;R=A/G  
`<()`可以创建一个临时文件，并传递给 faops 执行，执行完自动删除  
```
cd $HOME/faops/test
exp=$(printf ">read\n%s\n" ANNG)
res=$(faops filter -l 0 -N <(printf ">read\n%s\n" AMRG) stdout)
if [ "$exp" = "$res" ]; then                      
   echo "-N converts successfully"         
else                      
   echo "Failed"         
fi
```
## bats代码⑧:
```
@test "filter: remove dashes" {
    exp=$(printf ">read\n%s\n" ARG)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 -d <(printf ">read\n%s\n" A-RG) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑧:
`faops filter -l 0 -d`的`-d`参数可以用来删除序列中的破折号（-）  
```
cd $HOME/faops/test
exp=$(printf ">read\n%s\n" ARG)
res=$(faops filter -l 0 -d <(printf ">read\n%s\n" A-RG) stdout)
if [ "$exp" = "$res" ]; then                         
   echo "'-' has been removed successfully"         
else                         
   echo "Failed"         
fi
```
## bats代码⑨:
```
@test "filter: Upper cases" {
    exp=$(printf ">read\n%s\n" ATCG)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 -U <(printf ">read\n%s\n" AtcG) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑨:
`faops filter -l 0 -U`的`-U`参数可以将序列中的小写字母转换为大写字母  
```
cd $HOME/faops/test
exp=$(printf ">read\n%s\n" ATCG)    
res=$(faops filter -l 0 -U <(printf ">read\n%s\n" AtcG) stdout)
if [ "$exp" = "$res" ]; then                         
   echo "-U converts successfully"         
else                         
   echo "Failed"         
fi
```
## bats代码⑩:
```
@test "filter: simplify seq names" {
    exp=$(printf ">read\n%s\n" ANNG)
    res=$($BATS_TEST_DIRNAME/../faops filter -l 0 -s <(printf ">read.1\n%s\n" ANNG) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码⑩:
`faops filter -l 0 -s`的`-s`参数可以简化 FASTA 文件的描述行，移除序列名称中的后缀和额外描述  
```
cd $HOME/faops/test
exp=$(printf ">read\n%s\n" ANNG)    
res=$(faops filter -l 0 -s <(printf ">read.1\n%s\n" ANNG) stdout)
echo "exp:$exp; res:$res"
```
## bats代码11:
```
@test "filter: fastq to fasta" {
    run $BATS_TEST_DIRNAME/../faops filter $BATS_TEST_DIRNAME/test.seq stdout
    run bash -c "echo \"${output}\" | wc -l | xargs echo "
    assert_equal 6 "${output}"
}
```
## 可以运行的bash代码11:
利用`faops filter`可以将 fastq 格式文件转换为 fasta 格式文件  
```
cd $HOME/faops/test
res=$(faops filter test.seq stdout | wc -l | xargs echo)
echo "res:$res"
```
## bats代码12:
```
@test "faFilter: minSize" {
    if ! hash faFilter 2>/dev/null ; then
        skip "Can't find faFilter"
    fi

    exp=$(faFilter -minSize=10 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops filter -a 10 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码12:
利用`faops filter -a 10`可以过滤出长度≥10的序列  
• `-a`：最小序列长度阈值  
```
cd $HOME/faops/test
if ! hash faFilter 2>/dev/null ; then                 
   echo "Can't find faFilter"
else
exp=$(faFilter -minSize=10 ufasta.fa stdout | grep '^>')
res=$(faops filter -a 10 ufasta.fa stdout | grep '^>')
   if [ "$exp" = "$res" ]; then                   
      echo "faops_filter filters successfully"         
   else                   
      echo "Failed"         
   fi
fi
```
## bats代码13:
```
@test "faFilter: maxSize" {
    if ! hash faFilter 2>/dev/null ; then
        skip "Can't find faFilter"
    fi

    exp=$(faFilter -maxSize=50 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops filter -a 1 -z 50 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码13:
利用`faops filter -z 50`可以过滤出长度≤50的序列  
• `-z`：最大序列长度阈值
```
cd $HOME/faops/test
if ! hash faFilter 2>/dev/null ; then                    
   echo "Can't find faFilter"
else
exp=$(faFilter -maxSize=50 ufasta.fa stdout | grep '^>')
res=$(faops filter -a 1 -z 50 ufasta.fa stdout | grep '^>')
   if [ "$exp" = "$res" ]; then                         
      echo "faops_filter filters successfully"            
   else                         
      echo "Failed"            
   fi
fi
```
## bats代码14:
```
@test "faFilter: minSize maxSize" {
    if ! hash faFilter 2>/dev/null ; then
        skip "Can't find faFilter"
    fi

    exp=$(faFilter -minSize=10 -maxSize=50 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops filter -a 10 -z 50 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码14:
利用`faops filter -a 10 -z 50`可以过滤出长度在10和50之间的序列  
```
cd $HOME/faops/test
if ! hash faFilter 2>/dev/null ; then                    
   echo "Can't find faFilter"
else
exp=$(faFilter -minSize=10 -maxSize=50 ufasta.fa stdout | grep '^>')
res=$(faops filter -a 10 -z 50 ufasta.fa stdout | grep '^>')
   if [ "$exp" = "$res" ]; then                         
      echo "faops_filter filters successfully"            
   else                         
      echo "Failed"            
   fi
fi
```
## bats代码15:
```
@test "faFilter: uniq" {
    if ! hash faFilter 2>/dev/null ; then
        skip "Can't find faFilter"
    fi

    exp=$(faFilter -uniq <(cat $BATS_TEST_DIRNAME/ufasta.fa $BATS_TEST_DIRNAME/ufasta.fa) stdout | grep '^>')
    res=$($BATS_TEST_DIRNAME/../faops filter -u -a 1 <(cat $BATS_TEST_DIRNAME/ufasta.fa $BATS_TEST_DIRNAME/ufasta.fa) stdout | grep '^>')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码15:
利用`faops filter -u`可以去除序列中的重复序列  
```
cd $HOME/faops/test
if ! hash faFilter 2>/dev/null ; then                    
   echo "Can't find faFilter"
else
exp=$(faFilter -uniq <(cat ufasta.fa ufasta.fa) stdout | grep '^>')
res=$(faops filter -u -a 1 <(cat ufasta.fa ufasta.fa) stdout | grep '^>')
   if [ "$exp" = "$res" ]; then                         
      echo "faops_filter filters successfully"            
   else                         
      echo "Failed"            
   fi
fi
```
# 09-split-name.bats  
## bats代码①:
```
@test "split-name: all sequences" {
    mytmpdir=`mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir'`

    run bash -c "
        $BATS_TEST_DIRNAME/../faops split-name $BATS_TEST_DIRNAME/ufasta.fa $mytmpdir \
        && find $mytmpdir -name '*.fa' | wc -l | xargs echo
    "
    assert_equal 50 "${output}"

    rm -fr ${mytmpdir}
}
```
## 可以运行的bash代码①:
建立临时目录，将 ufasta.fa 文件中的多个序列按序列名拆分成多个单序列文件  
```
cd $HOME/faops/test
mytmpdir=$(mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir')
faops split-name ufasta.fa $mytmpdir
if [ $? -eq 0 ];then
   echo "Split successfully"
   ls $mytmpdir
else
   echo "Failed"
fi
file_count=$(find $mytmpdir -name '*.fa' | wc -l | xargs echo)
echo "$file_count"
rm -fr $mytmpdir
```
## bats代码②:
```
@test "split-name: size restrict" {
    mytmpdir=`mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir'`

    run bash -c "
        $BATS_TEST_DIRNAME/../faops filter -a 10 $BATS_TEST_DIRNAME/ufasta.fa stdout \
        | $BATS_TEST_DIRNAME/../faops split-name stdin $mytmpdir \
        && find $mytmpdir -name '*.fa' | wc -l | xargs echo
    "
    assert_equal 44 "${output}"

    rm -fr ${mytmpdir}
}
```
## 可以运行的bash代码②:
建立临时目录，将 ufasta.fa 文件中长度≥10的多个序列按序列名拆分成多个单序列文件
```
cd $HOME/faops/test
mytmpdir=$(mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir')
faops filter -a 10 ufasta.fa stdout | faops split-name stdin $mytmpdir
if [ $? -eq 0 ];then   
   echo "Split successfully"   
   ls $mytmpdir
else   
   echo "Failed"
fi
file_count=$(find $mytmpdir -name '*.fa' | wc -l | xargs echo)
echo "$file_count"
rm -fr $mytmpdir
```
# 10-split-about.bats
## bats代码①:
```
@test "split-about: 2000 bp " {
    mytmpdir=`mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir'`

    run bash -c "
        $BATS_TEST_DIRNAME/../faops split-about $BATS_TEST_DIRNAME/ufasta.fa 2000 $mytmpdir \
        && find $mytmpdir -name '*.fa' | wc -l | xargs echo
    "
    assert_equal 5 "${output}"

    rm -fr ${mytmpdir}
}
```
## 可以运行的bash代码①:
建立临时目录，利用`faops split-about ufasta.fa 2000 $mytmpdir`将 ufasta.fa 文件按每个 2000bp（约等于）拆分成多个包含几个序列的文件（单个序列不会被拆分），输出文件名为000.fa、001.fa......  
```
cd $HOME/faops/test
mytmpdir=$(mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir')
faops split-about ufasta.fa 2000 $mytmpdir
if [ $? -eq 0 ];then      
   echo "Split successfully"      
   ls $mytmpdir
else      
   echo "Failed"
fi
file_count=$(find $mytmpdir -name '*.fa' | wc -l | xargs echo)
echo "$file_count"
rm -fr $mytmpdir
```
## bats代码②:
```
@test "split-about: max parts" {
    mytmpdir=`mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir'`

    run bash -c "
        $BATS_TEST_DIRNAME/../faops split-about -m 2 $BATS_TEST_DIRNAME/ufasta.fa 2000 $mytmpdir \
        && find $mytmpdir -name '*.fa' | wc -l | xargs echo
    "
    assert_equal 2 "${output}"

    rm -fr ${mytmpdir}
}
```
## 可以运行的bash代码②:
建立临时目录，将 ufasta.fa 文件按每个 2000bp（约等于）拆分成多个包含几个序列的文件（单个序列不会被拆分）  
• `-m 2`会限制最多拆分文件数量为2，这样每个文件的碱基数量就不是2000，会根据最大文件数量进行调整  
```
cd $HOME/faops/test
mytmpdir=$(mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir')
faops split-about -m 2 ufasta.fa 2000 $mytmpdir
if [ $? -eq 0 ];then         
   echo "Split successfully"         
   ls $mytmpdir
else         
   echo "Failed"
fi
file_count=$(find $mytmpdir -name '*.fa' | wc -l | xargs echo)
echo "$file_count"
rm -fr $mytmpdir
```
## bats代码③:
```
@test "split-about: 2000 bp and size restrict" {
    mytmpdir=`mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir'`

    run bash -c "
        $BATS_TEST_DIRNAME/../faops filter -a 100 $BATS_TEST_DIRNAME/ufasta.fa stdout \
        | $BATS_TEST_DIRNAME/../faops split-about stdin 2000 $mytmpdir \
        && find $mytmpdir -name '*.fa' | wc -l | xargs echo
    "
    assert_equal 4 "${output}"

    rm -fr ${mytmpdir}
}
```
## 可以运行的bash代码③:
先利用`faops filter -a 100`过滤出长度≥100的序列，再利用`faops split-about`将过滤得到的序列按每个 2000bp 拆分成多个文件  
```
cd $HOME/faops/test
mytmpdir=$(mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir')
faops filter -a 100 ufasta.fa stdout | faops split-about stdin 2000 $mytmpdir
if [ $? -eq 0 ];then         
   echo "Split successfully"         
   ls $mytmpdir
else         
   echo "Failed"
fi
file_count=$(find $mytmpdir -name '*.fa' | wc -l | xargs echo)
echo "$file_count"
rm -fr $mytmpdir
```
## bats代码④:
```
@test "split-about: 1 bp" {
    mytmpdir=`mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir'`

    run bash -c "
        $BATS_TEST_DIRNAME/../faops split-about $BATS_TEST_DIRNAME/ufasta.fa 1 $mytmpdir \
        && find $mytmpdir -name '*.fa' | wc -l | xargs echo
    "
    assert_equal 50 "${output}"

    rm -fr ${mytmpdir}
}
```
## 可以运行的bash代码④:
利用`faops split-about`将 ufasta.fa 中的序列按每个 1bp 拆分成多个文件，由于每条序列不会被拆分，所以相当于按序列名拆分  
```
cd $HOME/faops/test
mytmpdir=$(mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir')
faops split-about ufasta.fa 1 $mytmpdir
if [ $? -eq 0 ];then         
   echo "Split successfully"         
   ls $mytmpdir
else         
   echo "Failed"
fi
file_count=$(find $mytmpdir -name '*.fa' | wc -l | xargs echo)
echo "$file_count"
rm -fr $mytmpdir
```
## bats代码⑤:
```
@test "split-about: 1 bp even" {
    mytmpdir=`mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir'`

    run bash -c "
        $BATS_TEST_DIRNAME/../faops split-about -e $BATS_TEST_DIRNAME/ufasta.fa 1 $mytmpdir \
        && find $mytmpdir -name '*.fa' | wc -l | xargs echo
    "
    assert_equal 26 "${output}"

    rm -fr ${mytmpdir}
}
```
## 可以运行的bash代码⑤:
利用`split-about -e`可以实现均匀拆分，将序列均匀分配到固定数量的文件中（总序列数/2）  
```
cd $HOME/faops/test
mytmpdir=$(mktemp -d 2>/dev/null || mktemp -d -t 'mytmpdir')
faops split-about -e ufasta.fa 1 $mytmpdir
if [ $? -eq 0 ];then         
   echo "Split successfully"         
   ls $mytmpdir
else         
   echo "Failed"
fi
file_count=$(find $mytmpdir -name '*.fa' | wc -l | xargs echo)
echo "$file_count"
rm -fr $mytmpdir
```
# 11-n50.bats  
## bats代码①:
```
@test "n50: display header" {
    run bash -c "$BATS_TEST_DIRNAME/../faops n50 $BATS_TEST_DIRNAME/ufasta.fa"
    assert_equal "N50${tab}314" "${lines[0]}"
}
```
## 可以运行的bash代码①:
利用`faops n50`可以计算 fasta 文件的 N50 的值，直接按“N50  N50值”格式输出  
N50：将所有序列按长度排序后，从长到短进行累计，累计长度达到实际组装总长度50%时的序列长度，值越大说明组装质量越好（长序列越多）  
```
cd $HOME/faops/test
N50=$(faops n50 ufasta.fa)
echo "$N50"
```
## bats代码②:
```
@test "n50: don't display header" {
    run $BATS_TEST_DIRNAME/../faops n50 -H $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | xargs echo "
    assert_equal "314" "${output}"
}
```
## 可以运行的bash代码②:
`faops n50 -H`可以直接输出 N50 的值，没有表头  
```
cd $HOME/faops/test
N50=$(faops n50 -H ufasta.fa | xargs echo)
echo "$N50"
```
## bats代码③:
```
@test "n50: set genome size (NG50)" {
    run $BATS_TEST_DIRNAME/../faops n50 -H -g 10000 $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | xargs echo "
    assert_equal "297" "${output}"
}
```
## 可以运行的bash代码③:
`faops n50 -H -g 10000`可以根据已知或预计基因组大小（如10000bp），计算 NG50 的值  
NG50：根据已知或预计基因组大小计算，是 N50 的变体  
```
cd $HOME/faops/test
N50=$(faops n50 -H -g 10000 ufasta.fa | xargs echo)
echo "$N50"
```
## bats代码④:
```
@test "n50: sum of size" {
    run $BATS_TEST_DIRNAME/../faops n50 -H -S $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | xargs echo "
    assert_equal "314 9317" "${output}"
}
```
## 可以运行的bash代码④:
利用`faops n50 -H -S`不仅可以计算 N50 的值，还可以计算序列的总长度（size）  
```
cd $HOME/faops/test
res=$(faops n50 -H -S ufasta.fa | xargs echo)
echo "$res"
```
## bats代码⑤:
```
@test "n50: sum and average of size" {
    run $BATS_TEST_DIRNAME/../faops n50 -H -S -A $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | xargs echo "
    assert_equal "314 9317 186.34" "${output}"
}
```
## 可以运行的bash代码⑤:
利用`faops n50 -H -S -A`不仅可以计算 N50 的值，还可以计算序列的总长度（size）和平均每条序列的长度（average）  
```
cd $HOME/faops/test
res=$(faops n50 -H -S -A ufasta.fa | xargs echo)
echo "$res"
```
## bats代码⑥:
```
@test "n50: E-size" {
    run $BATS_TEST_DIRNAME/../faops n50 -H -E $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | xargs echo "
    assert_equal "314 314.70" "${output}"
}
```
## 可以运行的bash代码⑥:
利用`faops n50 -H -E`不仅可以计算 N50 的值，还可以计算 E-size 的值  
E-size：是另一个组装质量指标，表示随机选择一个碱基时，它所在序列的期望长度（计算公式：∑(序列长度²) / 总碱基数），强调长序列的贡献，对长序列更敏感  
```
cd $HOME/faops/test
res=$(faops n50 -H -E ufasta.fa | xargs echo)
echo "$res"
```
## bats代码⑦:
```
@test "n50: n10" {
    run $BATS_TEST_DIRNAME/../faops n50 -H -N 10 $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | xargs echo "
    assert_equal "516" "${output}"
}
```
## 可以运行的bash代码⑦:
利用`faops n50 -H -N 10`可以计算 N10 的值  
N10：将所有序列按长度排序后，累计长度达到总长度10%时的序列长度，相比N50更关注最长的那部分序列  
```
cd $HOME/faops/test
res=$(faops n50 -H -N 10 ufasta.fa | xargs echo)
echo "$res"
```
## bats代码⑧:
```
@test "n50: n90 with header" {
    run $BATS_TEST_DIRNAME/../faops n50 -N 90 $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | xargs echo "
    assert_equal "N90 112" "${output}"
}
```
## 可以运行的bash代码⑧:
利用`faops n50 -N 90`可以计算 N90 的值，显示表头  
N90：将所有序列按长度排序后，累计长度达到总长度90%时的序列长度，反映较短序列的长度特征  
```
cd $HOME/faops/test
res=$(faops n50 -N 90 ufasta.fa | xargs echo)
echo "$res"
```
## bats代码⑨:
```
@test "n50: only count of sequences" {
    run $BATS_TEST_DIRNAME/../faops n50 -N 0 -C $BATS_TEST_DIRNAME/ufasta.fa
    run bash -c "echo \"${output}\" | xargs echo "
    assert_equal "C 50" "${output}"
}
```
## 可以运行的bash代码⑨:
`faops n50 -N 0 -C`中`-N 0`表示不进行N统计计算，`-C` 只统计序列的数量  
```
cd $HOME/faops/test
res=$(faops n50 -N 0 -C ufasta.fa | xargs echo)
echo "$res"
```
# 12-order.bats
## bats代码①:
```
@test "order: inline names" {
    exp=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -A 1 '^>read12')
    res=$($BATS_TEST_DIRNAME/../faops order -l 0 $BATS_TEST_DIRNAME/ufasta.fa <(echo read12) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码①:
利用`faops order -l 0`提取 fasta 文件中的指定序列  
```
cd $HOME/faops/test
exp=$(faops filter -l 0 ufasta.fa stdout | grep -A 1 '^>read12')
res=$(faops order -l 0 ufasta.fa <(echo read12) stdout)
if [ "$exp" = "$res" ];then
   echo "faops_order extracts sequence correctly"
else
   echo "Failed"
fi
```
## bats代码②:
```
@test "order: correct orders" {
    exp=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -A 1 '^>read12')
    exp+=$'\n'
    exp+=$($BATS_TEST_DIRNAME/../faops filter -l 0 $BATS_TEST_DIRNAME/ufasta.fa stdout | grep -A 1 '^>read5')
    res=$($BATS_TEST_DIRNAME/../faops order -l 0 $BATS_TEST_DIRNAME/ufasta.fa <(echo read12 read5) stdout)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码②:
利用`faops order -l 0`加`<(echo read12 read5)`可以按指定顺序提取相应序列（如先 read12 后 read5）  
```
cd $HOME/faops/test
exp=$(faops filter -l 0 ufasta.fa stdout | grep -A 1 '^>read12')    
exp+=$'\n'    
exp+=$(faops filter -l 0 ufasta.fa stdout | grep -A 1 '^>read5')    
res=$(faops order -l 0 ufasta.fa <(echo read12 read5) stdout)
if [ "$exp" = "$res" ];then
   echo "faops_order extracts sequences correctly"
else
   echo "Failed"
fi
```
## bats代码③:
```
@test "order: compare with some" {
    exp=$($BATS_TEST_DIRNAME/../faops order $BATS_TEST_DIRNAME/ufasta.fa \
            <($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa | sort -n -r -k2,2 | cut -f 1) \
            stdout )

    res=$( for word in $($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa | sort -n -r -k2,2 | cut -f 1); do
            $BATS_TEST_DIRNAME/../faops some $BATS_TEST_DIRNAME/ufasta.fa <(echo ${word}) stdout
        done )

    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码③:
用`order`和`some`两种方法，结合`size`按序列长度从大到小的顺序提取全部序列  
`order`可以直接一步提取，而`some`需要借助循环分布提取  
```
cd $HOME/faops/test
exp=$(faops order ufasta.fa <(faops size ufasta.fa | sort -n -r -k2,2 | cut -f 1) stdout )
echo "$exp"
list=$(faops size ufasta.fa | sort -n -r -k2,2 | cut -f 1)
res=""
for word in $list; do
    sequence=$(faops some ufasta.fa <(echo $word) stdout)
    res+="$sequence"$'\n'        
done
echo "$res"
```
# 13-replace.bats
## bats代码①:
```
@test "replace: inline names" {
    exp=">428"
    res=$($BATS_TEST_DIRNAME/../faops replace $BATS_TEST_DIRNAME/ufasta.fa \
        <(printf "%s\t%s\n" read12 428) stdout \
        | grep '^>428')
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码①:
利用`faops replace`对指定序列名进行替换，输出全部序列  
```
cd $HOME/faops/test
exp=">428"    
res=$(faops replace ufasta.fa <(printf "%s\t%s\n" read12 428) stdout | grep '^>428')
echo "exp:$exp;res:$res"
```
## bats代码②:
```
@test "replace: -s" {
    exp="7"
    res=$($BATS_TEST_DIRNAME/../faops replace -s $BATS_TEST_DIRNAME/ufasta.fa \
        <(printf "%s\t%s\n" read12 428) stdout |
        wc -l |
        xargs echo)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码②:
利用`faops replace -s`只输出被替换的指定序列的描述行和序列信息  
```
cd $HOME/faops/test
exp="7"    
res=$(faops replace -s ufasta.fa <(printf "%s\t%s\n" read12 428) stdout | wc -l | xargs echo)
echo "exp:$exp;res:$res"
```
## bats代码③:
```
@test "replace: with replace.tsv" {
    exp=$(cat $BATS_TEST_DIRNAME/replace.tsv | cut -f 2)
    res=$($BATS_TEST_DIRNAME/../faops replace $BATS_TEST_DIRNAME/ufasta.fa \
        $BATS_TEST_DIRNAME/replace.tsv stdout \
        | grep '^>' | grep -v 'read' | sed 's/>//' )
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码③:
借助映射文件 replace.tsv ，利用`faops replace`实现同时替换多条序列（只替换 replace.tsv 中提到的序列，其余不变）  
`grep '^>'`找出所有描述行；`grep -v 'read'`去除所有含有 read 的未被替换的描述行；`sed 's/>//'`去除“>”只保留序列名  
```
cd $HOME/faops/test
exp=$(cat replace.tsv | cut -f 2)
res=$(faops replace ufasta.fa replace.tsv stdout | grep '^>' | grep -v 'read' | sed 's/>//' )
if [ "$exp" = "$res" ];then   
   echo "faops_replace replaces sequences name correctly"
else   
   echo "Failed"
fi
```
# 14-dazz.bats
## bats代码①:
```
@test "dazz: empty seqs count" {
    exp=$($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa | grep "\s0" | wc -l | xargs echo)
    res=$($BATS_TEST_DIRNAME/../faops dazz -a $BATS_TEST_DIRNAME/ufasta.fa stdout | grep "0_0" | wc -l | xargs echo)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码①:
利用`faops size`和`faops dazz`都可以计算序列长度为 0 的序列的数量  
`faops size`可以输出序列名+序列长度，`grep "\s0"`中 \s 可以匹配空白字符（空格、制表符等），grep "\s0" 可以匹配“空白字符+0”，即序列长度为 0 的行  
`faops dazz -a ufasta.fa stdout`后描述行会以 “>read/x/0_y” 格式输出（序列信息也会输出），x 为序列的序数（即文件中的第几个序列），y 为序列的长度，“0_0”表示序列长度为 0  
```
cd $HOME/faops/test
exp=$(faops size ufasta.fa | grep "\s0" | wc -l | xargs echo)    
res=$(faops dazz -a ufasta.fa stdout | grep "0_0" | wc -l | xargs echo)
echo "exp:$exp"
echo "res:$res"
```
## bats代码②:
```
@test "dazz: deduplicate seqs" {
    exp=$($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa $BATS_TEST_DIRNAME/ufasta.fa.gz | grep "\s0" | wc -l | xargs echo)
    res=$(gzip -d -c -f $BATS_TEST_DIRNAME/ufasta.fa $BATS_TEST_DIRNAME/ufasta.fa.gz | $BATS_TEST_DIRNAME/../faops dazz stdin stdout | grep "0_0" | wc -l | xargs echo)
    assert_equal "$(($exp / 2))" "$res"
}
```
## 可以运行的bash代码②:
`faops dazz`可以去重序列  
`gzip -d -c -f`中 -d 表示解压缩；-c 表示写到标准输出； -f 表示强制执行  
```
cd $HOME/faops/test
exp=$(faops size ufasta.fa ufasta.fa.gz | grep "\s0" | wc -l | xargs echo)    
res=$(gzip -d -c -f ufasta.fa ufasta.fa.gz | faops dazz stdin stdout | grep "0_0" | wc -l | xargs echo)
echo "exp:$(($exp / 2))"
echo "res:$res"
```
## bats代码③:
```
@test "dazz: duplicated seqs" {
    exp=$($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa $BATS_TEST_DIRNAME/ufasta.fa.gz | grep "\s0" | wc -l | xargs echo)
    res=$(gzip -d -c -f $BATS_TEST_DIRNAME/ufasta.fa $BATS_TEST_DIRNAME/ufasta.fa.gz | $BATS_TEST_DIRNAME/../faops dazz -a stdin stdout | grep "0_0" | wc -l | xargs echo)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码③:
`faops dazz -a`不会进行去重，会保留所有的序列信息  
```
cd $HOME/faops/test
exp=$(faops size ufasta.fa ufasta.fa.gz | grep "\s0" | wc -l | xargs echo) 
res=$(gzip -d -c -f ufasta.fa ufasta.fa.gz | faops dazz -a stdin stdout | grep "0_0" | wc -l | xargs echo)
echo "exp:$exp"
echo "res:$res"
```
# 15-interleave.bats
## bats代码①:
```
@test "interleave: empty seqs count" {
    exp=$($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa | grep "\s0" | wc -l | xargs echo)
    res=$($BATS_TEST_DIRNAME/../faops interleave $BATS_TEST_DIRNAME/ufasta.fa $BATS_TEST_DIRNAME/ufasta.fa.gz | grep "^$" | wc -l | xargs echo)
    assert_equal "$(($exp * 2))" "$res"
}
```
## 可以运行的bash代码①:
`faops interleave`可以交错合并两个文件，总序列数会翻倍  
输出为：  
>read1/1  
序列信息  
>read1/2  
序列信息  
......  
```
cd $HOME/faops/test
exp=$(faops size ufasta.fa | grep "\s0" | wc -l | xargs echo)    
res=$(faops interleave ufasta.fa ufasta.fa.gz | grep "^$" | wc -l | xargs echo)
echo "exp:$(($exp * 2))"
echo "res:$res"
```
## bats代码②:
```
@test "interleave: empty seqs count (single)" {
    exp=$($BATS_TEST_DIRNAME/../faops size $BATS_TEST_DIRNAME/ufasta.fa | grep "\s0" | wc -l | xargs echo)
    res=$($BATS_TEST_DIRNAME/../faops interleave $BATS_TEST_DIRNAME/ufasta.fa | grep "^$" | wc -l | xargs echo)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码②:
`faops interleave`可以交错合并两个文件，当只有一个文件时，会输出N  
输出为：  
>read1/1  
序列信息  
>read1/2  
>N  
......  
```
cd $HOME/faops/test
exp=$(faops size ufasta.fa | grep "\s0" | wc -l | xargs echo)    
res=$(faops interleave ufasta.fa | grep "^$" | wc -l | xargs echo)
echo "exp:$exp"
echo "res:$res"
```
## bats代码③:
```
@test "interleave: fq" {
    run bash -c "
        $BATS_TEST_DIRNAME/../faops interleave -q $BATS_TEST_DIRNAME/R1.fq.gz $BATS_TEST_DIRNAME/R2.fq.gz |
            grep '^!$' |
            wc -l
    "
    assert_equal 0 "${output}"
}
```
## 可以运行的bash代码③:
利用`faops interleave -q`交错合并两个 fastq 文件  
输出为：  
@read1/1  
序列及质量信息  
@read1/2  
序列及质量信息  
......  
合并后不会产生无效的质量值，即包含单个感叹号 ! 的行  
```
cd $HOME/faops/test
faops interleave -q R1.fq.gz R2.fq.gz | grep '^!$' | wc -l
```
## bats代码④:
```
@test "interleave: fq (single)" {
    run bash -c "
        $BATS_TEST_DIRNAME/../faops interleave -q $BATS_TEST_DIRNAME/R1.fq.gz |
            grep '^!$' |
            wc -l
    "
    assert_equal 25 "${output}"
}
```
## 可以运行的bash代码④:
利用`faops interleave -q`交错合并两个 fastq 文件,当只有一个文件时,系列质量会输出 !  
输出为：  
@read1/1  
序列及质量信息  
@read1/2  
N及!  
......  
```
cd $HOME/faops/test
faops interleave -q R1.fq.gz | grep '^!$' | wc -l
```
# 16-region.bats
## bats代码①:
```
@test "region: from file" {
    exp=$(cat $BATS_TEST_DIRNAME/region.txt | wc -l | xargs echo)
    res=$($BATS_TEST_DIRNAME/../faops region -l 0 $BATS_TEST_DIRNAME/ufasta.fa $BATS_TEST_DIRNAME/region.txt stdout | wc -l | xargs echo)
    assert_equal "$(($exp * 2))" "$res"
}
```
## 可以运行的bash代码①:
借助 region.txt 文件可以从 ufasta.fa 文件中提取指定序列的指定位置的碱基  
如：region.txt 中的内容为  
read0:1-10  
read12:50-60  
利用`faops region -l 0`可以提取 read0 的第 1-10 个碱基、read12 的第 50-60 个碱基  
输出为  
>read0:1-10  
tCGTTTAACC  
>read12:50-60  
TtgTgtcACag  
```
cd $HOME/faops/test
exp=$(cat region.txt | wc -l | xargs echo)    
res=$(faops region -l 0 ufasta.fa region.txt stdout | wc -l | xargs echo)
echo "exp:$(($exp * 2))"
echo "res:$res"
```
## bats代码②:
```
@test "region: frag" {
    exp=$($BATS_TEST_DIRNAME/../faops frag $BATS_TEST_DIRNAME/ufasta.fa 1 10 stdout)
    res=$($BATS_TEST_DIRNAME/../faops region -l 0 $BATS_TEST_DIRNAME/ufasta.fa <(echo read0:1-10) stdout)
    assert_equal "${exp}" "${res}"
}
```
## 可以运行的bash代码②:
利用`faops region -l 0`和`faops frag`都可以提取一条序列的指定位置的碱基  
```
cd $HOME/faops/test
exp=$(faops frag ufasta.fa 1 10 stdout)    
res=$(faops region -l 0 ufasta.fa <(echo read0:1-10) stdout)
echo "exp:$exp"
echo "res:$res"
```
## bats代码③:
```
@test "region: 1 base" {
    exp=$($BATS_TEST_DIRNAME/../faops frag $BATS_TEST_DIRNAME/ufasta.fa 10 10 stdout)
    res=$($BATS_TEST_DIRNAME/../faops region -l 0 $BATS_TEST_DIRNAME/ufasta.fa <(echo read0:10) stdout)
    assert_equal "${exp}" "${res}"
}
```
## 可以运行的bash代码③:
利用`faops region -l 0`和`faops frag`都可以提取一条序列的指定位置的 1 个碱基  
```
cd $HOME/faops/test
exp=$(faops frag ufasta.fa 10 10 stdout)    
res=$(faops region -l 0 ufasta.fa <(echo read0:10) stdout)
echo "exp:$exp"
echo "res:$res"
```
## bats代码④:
```
@test "region: strand" {
    exp=$(echo -e ">read0(+):10\nC")
    res=$($BATS_TEST_DIRNAME/../faops region -s -l 0 $BATS_TEST_DIRNAME/ufasta.fa <(echo read0:10) stdout)
    assert_equal "${exp}" "${res}"
}
```
## 可以运行的bash代码④:
利用`faops region -s -l 0`可以提取一条序列的指定位置的碱基，`-s`启用链向处理，提取正链序列  
输出为  
>read0(+):10  
C  
```
cd $HOME/faops/test
exp=$(echo -e ">read0(+):10\nC")    
res=$(faops region -s -l 0 ufasta.fa <(echo read0:10) stdout)
echo "exp:$exp"
echo "res:$res"
```
## bats代码⑤:
```
@test "region: regions" {
    exp=4
    res=$($BATS_TEST_DIRNAME/../faops region -l 0 $BATS_TEST_DIRNAME/ufasta.fa <(echo read1:1-10,50-60) stdout | wc -l | xargs echo)
    assert_equal "${exp}" "${res}"
}
```
## 可以运行的bash代码④:
利用`faops region -l 0`可以同时提取一条序列的不同位置的碱基  
输出为  
>read1:1-10  
taGGCGcGGg  
>read1:50-60  
TacgtaACatc  
```
cd $HOME/faops/test
exp=4    
res=$(faops region -l 0 ufasta.fa <(echo read1:1-10,50-60) stdout | wc -l | xargs echo)
echo "exp:$exp"
echo "res:$res"
```
# 17-masked.bats
## bats代码①:
```
@test "masked 1" {
    exp="read46:3-4"
    res=$($BATS_TEST_DIRNAME/../faops masked $BATS_TEST_DIRNAME/ufasta.fa | grep '^read46' | head -n 1)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码①:
利用`faops masked`可以显示所有序列所有被 masked 的位置（小写字母）  
```
cd $HOME/faops/test
exp="read46:3-4"
res=$(faops masked ufasta.fa | grep '^read46' | head -n 1)
echo "exp:$exp"
echo "res:$res"
```
## bats代码②:
```
@test "masked 2" {
    exp="read0:1"
    res=$($BATS_TEST_DIRNAME/../faops masked $BATS_TEST_DIRNAME/ufasta.fa | grep '^read0' | head -n 1)
    assert_equal "$exp" "$res"
}
```
## 可以运行的bash代码②:
利用`faops masked`可以显示所有序列所有被 masked 的位置（小写字母），同 ①  
```
cd $HOME/faops/test
exp="read0:1"    
res=$(faops masked ufasta.fa | grep '^read0' | head -n 1)
echo "exp:$exp"
echo "res:$res"
```

