## awk

1. 以 / 为分隔符，获取第2个，并且去掉最后一个字符

   ```bash
   awk -F '/' '{gsub('/.$/','',$2),print $2}' FILENAME	
   
   #	[kworke/1:2] -> 1:2
   ```

2. 从第二行开始获取

   ```bash
   awk 'NR>=2 {print $1}' FILENAME
   ```

3. 获取文件中按照 : 分割的第三项等于0的行，并打印第一项

   ```shell
   awk '($3==0) {print $1}' /etc/passwd
   
   # awk '(pattern) { action }' file
   ```

4. 从第二行开始，将第四列的内容叠加

   ```
   awk 'NR > 1{sum += $4} END {print sum}'
   ```

   





## sed

1. 从环境变量中获取密码，将文件中的密码行(行首为password)替换成新密码行，原地替换

   ```bash
   sed -i '/^password/c\password '"$PWD"'' config
   # 当在sed中引用环境变量时，先使用""将$PWD转义为实际的值，再使用
   ```

   

## grep

1. 判断是否满足密码复杂度：最少10位、至少一个大小写字母，至少一个特殊字符。

   ```bash
   echo "$PWD" | grep -P '^(?=.*[a-z])(?=.*[A-Z])(?=.*[!@#,.]).{10,}$'
   # grep 中的 -P 表示使用Perl格式的正则表达式
   ```

   
