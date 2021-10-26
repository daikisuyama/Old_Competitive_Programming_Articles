# 感想

B問題を**不注意で読み違えました**…。  
C問題は部分列なのでDPを用いようとしたのですが、B問題の焦りで正確な考察ができませんでした。

# [A問題](https://yukicoder.me/problems/no/1236)

$x$時$y$秒の時に長針と短針が重なるとすれば、以下の式が成り立ちます。

```math
\begin{align}
&\frac{y}{60} \times 360  \div 60=\frac{x}{12} \times 360 + \frac{y}{60} \times 30 \div 60 \\
&\leftrightarrow y=60 \times 60 \times x \div 11
\end{align}
```

また、$x$は12の余りで考えればよいので$x=$0~11であり、上記の式に従えば**切り捨てた値として**$y$が求められるので、与えられた$x$に対して与えられた$y$の値が$x$時台に重なる時刻より前かどうかを考えれば良いです。

```python:A.py
cand=[60*i*60//11 for i in range(12)]
a,b=map(int,input().split())
a%=12
b*=60
if cand[a]>=b:
    print(int(cand[a]-b))
else:
    a=(a+1)%12
    print(int(cand[a]+3600-b))
```

#[B問題](https://yukicoder.me/problems/no/1237)

問題を良く読むと**$10^9+7$を答えで割った余りを求める**ので、$A\_k \geqq 4$の時に$A\_k\^{A\_k !}\>10\^9+7$となり、余りは$10^9+7$になります。また、掛け算の際に**$A\_k=0$の場合は他の数が任意の数で答えは0**になります。よって、$A\_{min}=0$のときは-1を出力します($A\_{min} \neq 0$のときは答えは0にはならず、割り算が行えます。)。

以上より、$A\_{min}=0$のときは-1を出力、$A_{max}\>3$のときは$10^9+7$を出力、それ以外の場合は愚直に計算をそれぞれ行うことで答えは以下のようになります。

```python:B.py
n=int(input())
a=list(map(int,input().split()))

mod=10**9+7
if min(a)==0:
    print(-1)
    exit()
if max(a)>3:
    print(mod)
    exit()
ans=1
for i in a:
    sub=1
    for j in range(i):
        sub*=(j+1)
    ans*=(i**sub)
    if ans>10**9+7:
        print(mod)
        exit()
print(mod%ans)
```

# [C問題](https://yukicoder.me/problems/no/1238)

部分列なので、**$dp[i]:=$($i$番目までの集合の部分集合に対しての何か)とし遷移を含むか含まないかの2通りとする**のが典型的なパターンです。

今回は平均が$k$以上で平均は人数によるので、得点と人数の両方の情報を持ちながらのDPが必要そうです。しかし、計算量を考えれば両方を持つのが難しくここでは**人数の情報を削ること**を考えます。つまり、**それぞれの人の得点を予め$-k$しておく**ことで部分集合に含まれる**生徒の得点の合計が0以上であるか**を調べることにします。よって、**部分集合の点数がその集合の要素数によらない条件**として情報を持つことができたので、あとは以下のようにDPをおくだけです。

$dp[i][j]:=$($i$番目までの集合の部分集合の合計の得点が$j$となる場合の数)

また、$-k$することで$j$が負になる可能性があるので、**最小値になりうる10000のぶんだけ下駄を履かせれば**以下のようなDPとなります。

$dp[i][j]:=$($i$番目までの集合の部分集合の合計の得点が$j-10000$となる場合の数)

そして、遷移は以下のようになります。

(1)$i$番目の要素を選ばない時
$dp[i][j]+=dp[i-1][j]$

(2)$i$番目の要素を選ぶ時
$dp[i][j+a[i]]+=dp[i-1][j]$
ただし、$0 \leqq j+a[i] \leqq 20000$
また、$i$番目の要素だけの集合の場合があり、$dp[i][a[i]+10000]+=1$

以上の遷移を行って、最終的に求めたいのは合計が0以上の場合の場合の数なので、$sum(dp[n-1][10000:])$となります。また、**求めるのは$10\^9+7$で割った余り**であることにも注意が必要です。

```python:C.py
mod=10**9+7
n,k=map(int,input().split())
a=[i-k for i in list(map(int,input().split()))]
dp=[[0]*20001 for i in range(n)]
dp[0][a[0]+10000]=1
for i in range(1,n):
    for j in range(20001):
        dp[i][j]=dp[i-1][j]
    dp[i][a[i]+10000]+=1
    for j in range(20001):
        dp[i][j]%=mod
        if 0<=j+a[i]<=20000:
            dp[i][j+a[i]]+=dp[i-1][j]
#和(マイナスインデックスも)
print(sum(dp[-1][10000:])%mod)
```