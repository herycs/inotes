# 创建文件

### 重定向

\> a.txt

### touch

touch a.txt

更新文件修改时间为当前时间

### echo

echo "hello world" > a.txt：复写文件

echo "friend!" >> a.txt：追加文件

### printf

同echo

### cat

cat > a.txt (ctrl + D | C结束输入文本)

### vi | vim

### nano

nano a.txt

### head

head -c 0k /dev/zero > a.txt

### tail

head -c 0k /dev/zero > a.txt

### truncate

文件缩小或者扩展为一个新的大小

truncate -s 0k a.txt