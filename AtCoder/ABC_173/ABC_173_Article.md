# 感想

今回は十分な考察ができないままコンテストに臨んだため、悪いパフォーマンスとなりました。

# [A問題](https://atcoder.jp/contests/abc173/tasks/abc173_a)

（不備があったので、省略します。）

```python:A.py
n=int(input())
if n%1000==0:
    print(0)
else:
    print(1000-n%1000)
```

# [B問題](https://atcoder.jp/contests/abc173/tasks/abc173_b)

（不備があったので、省略します。）

```python:B.py
n=int(input())
d={"AC":0,"WA":0,"TLE":0,"RE":0}
for i in range(n):
    d[input()]+=1
print("AC x "+str(d["AC"]))
print("WA x "+str(d["WA"]))
print("TLE x "+str(d["TLE"]))
print("RE x "+str(d["RE"]))
```


# [C問題](https://atcoder.jp/contests/abc173/tasks/abc173_c)

それぞれの赤で塗る行と列の選び方を考えたいので、入力で受け取った行列をコピーして赤(x)で上書きしていき、最終的に残った黒(#)の数を数えます。

```python:C.py
h,w,K=map(int,input().split())
c=[input() for i in range(h)]
ans=0
for i in range(2**h):
    for j in range(2**w):
        d=[[c[z][v] for v in range(w)]for z in range(h)]
        for k in range(h):
            if (i>>k)&1:
                for l in range(w):
                    d[k][l]="x"
        for k in range(w):
            if (j>>k)&1:
                for l in range(h):
                    d[l][k]="x"
        cnt=0
        for k in range(h):
            for l in range(w):
                cnt+=(d[k][l]=="#")
        ans+=(cnt==K)
print(ans)
```


# [D問題](https://atcoder.jp/contests/abc173/tasks/abc173_d)

まず、フレンドリーさの低い人を先に到着させても状況が良くならないため、**フレンドリーさの高い人から順に**到着した方が良いことが示せそうです。

また、この仮定のもとで、どの位置に割り込むべきかは今ある中で最大の位置ではないかという**貪欲法**を考えることができます。つまり、以下のように割り込むことができます。

![](/AtCoder/ABC_173/ABC_173_1.png)

上図では、その人のフレンドリーさを左側に心地よさを右側に書いています。このように割り込む時、$A_i$は$A_{[\frac{i+1}{2}]}$のフレンドリーさを得ることができます。これを実装して下記のようになります。

```python:D.py
n=int(input())
a=list(map(int,input().split()))
a.sort(reverse=True)
ans=0
for i in range(1,n):
    ans+=a[i//2]
print(ans)
```

# [E問題](https://atcoder.jp/contests/abc173/tasks/abc173_e)

まず、**$K$個の数の積が負となるパターンが少なそう**である事に気づいて負の数である場合を先に計算することを考えます。すると、$K$個の数の積が負となるのは$K$個のうち**負の数を奇数個しか選べない時**となります。ここで、0以上の数が一つ以上存在すれば、その数を含むか含まないかで$K$個の数に含まれる負の数の個数の偶奇を制御できます。したがって、$K$個の数の積が負となるのは、与えられた$N$個の数が全て負かつ$K$が奇数である場合(①)、$N=K$かつ与えられた$N$個の数に負の数が奇数個含まれる場合(②)、のどちらかになります。

①の考察↓
与えられた$N$個の数が全てが負または全てが0以上である場合の計算は難しくないので、まとめて計算しました。つまり、与えられた$N$個の数が全て負の数の時、$K$が偶数である場合は積が正なので絶対値の大きいものから順に$K$個の積が最大で、$K$が奇数である場合は積が負なので絶対値の小さいものから順に$K$個の積が最大です。また、全てが0以上の数の時も絶対値の大きいものから順に$K$個の積が最大です。

①の実装↓
0以上の数を`ap`に格納し、負の数を`am`に格納し、`len(ap)==0`または`len(am)==0`場合は$N$個の数が全て負の数(or全て正の数)となるので、考察の通りの実装をします。また、`ap`と`am`は絶対値の降順になるようにソートしておきます。

②の考察及び実装↓
$N=K$となる場合は与えられた$N$個の数の積を考えれば良いです。これは入力を受け取った直後に求めておきます。

以下では、$K$個の数の積の最大値が0以上であるもとでの方針を考えます。

まず、**負の数は偶数個**ないと$K$個の積が0以上にならないので、**絶対値の大きい数から順にペア**にして積を求めます。また、負の数と0以上の数の選択をするために0以上の数も大きい数から順にペアにして積を求め、これらを`apm2`に降順で格納します。(この後で0以上の数をいくつ選んだかをチェックしたいので、0以上の数の場合は1,負の数の場合は0のマークをつけておきます。)

ここで、ペアにしていった時に負の数も0以上の数も一つ余る可能性がありますが、負の数を選ぶ場合は積が負になるので選ぶ必要がありません。ただ、正の数は選ぶ可能性があるので、注意が必要です。

まず、**$K$が偶数の時**は`apm2`の前から$[\frac{k}{2}]$個の数の積を求めれば良いですが、**$K$が奇数の時**は$[\frac{k}{2}]$個の数に加えてもう一つ数を選択しなければなりません。この時、「まだ選んでない正の数で最大の数を一つ選ぶ(✳︎1)」とすると以下のような場合に最適ではありません。

```
5 3
-5 -1 1 2 3
```

上記の場合、-5,-1,3が最適ですが、(✳︎1)に従うと1,2,3が最適になります。つまり、「選んだ正の数で最小の数を除いて`apm2`の選んでない中で最大の組を選ぶ(✳︎2)」場合が最適となる可能性もあります。

よって、正の数で次に選ぶ数の`ap`中のインデックスを`p`に記録しながら、前から$[\frac{k}{2}]$個の数をまずは`ans`に`append`していきます(**除く操作が面倒なので積を求めずに一旦配列に格納する**ようにしました。)。

このもとで、$K$が偶数の場合は`ans`の全ての数の積を求め、$K$が奇数の場合は先ほどの(✳︎1)及び(✳︎2)に従えば`ap[p]`を選ぶ場合(①)、`ap[p-1]`を除いて`apm2[k//2][0]`を選ぶ場合(②)のどちらかを考えれば良いです。

ここで、`p==len(p)`の時は全ての0以上の数を使い切っているのでpに相当する数が存在せず、①のパターンは存在しません。また、`p==0`の時はまだ一つも0以上の数の組を使ってないので、②のパターンは存在しません。

以上を注意深く実装すると以下のようになります。また、与えられた配列の積を$10^9+7$で割った値を求める関数(`multarray`)を定義して実装を簡単にしました。

```python:E.py
#x:配列
mod=10**9+7
def multarray(x):
    ret=1
    for i in x:
        ret*=i
        ret%=mod
    return ret

n,k=map(int,input().split())
a=list(map(int,input().split()))

#n==kの場合はかけるだけ
if n==k:
    print(multarray(a))
    exit()

#0以上の数と負の数に分けて格納
ap,am=[],[]
for i in range(n):
    if a[i]>=0:
        ap.append(a[i])
    else:
        am.append(a[i])
#それぞれ絶対値の順番にソート
ap.sort(reverse=True)
am.sort()

#0以上の数のみか正の数のみの場合は先に場合分け
if len(am)==0:
    print(multarray(ap[:k]))
    exit()
if len(ap)==0:
    if k%2==0:
        print(multarray(am[:k]))
    else:
        print(multarray(am[::-1][:k]))
    exit()

#組にして格納する(0以上か負かを記録しておく)
apm2=[]
for i in range(len(am)//2):
    apm2.append((am[2*i]*am[2*i+1],0))
for i in range(len(ap)//2):
    apm2.append((ap[2*i]*ap[2*i+1],1))
apm2.sort(reverse=True)

#0以上の数で次にどこを見るのかをチェックしておく
p=0
ans=[]
for i in range(k//2):
    p+=(2*apm2[i][1])
    ans.append(apm2[i][0])

#Kが偶数の時
if k%2==0:
    print(multarray(ans))
    exit()

ans_cand=[]
#一個増やす場合
if p!=len(ap):
    ans_cand.append(multarray(ans)*ap[p]%mod)
#一個減らして二個増やす場合
if p!=0:
    #減らすindex
    check_i=ans.index(ap[p-1]*ap[p-2])
    #ap[p-1]を減らす操作(0除算を避ける)
    ans[check_i]=ap[p-2]
    ans.append(apm2[k//2][0])
    ans_cand.append(multarray(ans))
print(max(ans_cand))
```



# [F問題](https://atcoder.jp/contests/abc173/tasks/abc173_f)

まず、連結成分の個数を言い換えるますが、**森(木を含む)では連結成分の個数=頂点の数-辺の数(✳︎)が成り立ちます**。これは、木の場合は$1=N-(N-1)$でそれぞれの頂点が連結成分となる場合は$N=N-0$であり、森に含まれる辺を一つ除くとその辺を含む木が二分割されることで連結成分の個数が一つ増えその分割された木どうしも木になることから示せます。

このもとで、$\sum_{L=1}^{N} \sum_{R=L}^{N} f(L,R)$を計算しますが、この和の計算を一つずつ行うと間に合いません。また、(✳︎)よりそれぞれの$(L,R)$に対しての(頂点の数-辺の数)を計算すれば良く、**頂点の数と辺の数は独立に計算できる**ので、**頂点の数(1~n)が和の中に何回現れるのか**と**それぞれの辺が和の中に何回現れるのか**を考えれば良いです。

(1)頂点の数
頂点の数を$i$とした時、$(L,R)=(1,i),(2,i+1),…,(N-i+1,N)$の$N-i+1$回だけ和の中に現れるので、その和は$(N-i+1)*i$となります。したがって、任意の$i$についてこの値の和を求めれば良いです。

(2)辺の数
ある辺が$l,r(1 \leqq l \leqq r \leqq N)$を結んでいた時、$1 \leqq L \leqq l$かつ$r \leqq R \leqq N$であればその辺は部分グラフに含まれるので、その辺は和の中に$l \times (N-r+1)$回だけ現れます。したがって、任意の辺についてこの値の和を求めれば良いです。

以上より、(1)の和-(2)の和を求めれば答えとなります。

```python:F.py
n=int(input())
vertex=[(1+n-i)*(n-i)//2 for i in range(n)]
side=[0]*(n-1)
for i in range(n-1):
    l,r=list(map(int,input().split()))
    if l>r:
        l,r=r,l
    side[i]=l*(n-r+1)
print(sum(vertex)-sum(side))
```
 
----

以下は、F問題を解く際に重要な概念についてまとめたものです。

✳︎森
閉路を持たない(連結とは限らない)グラフのこと。木の集合と捉えるとわかりやすい。

✳︎道
ある頂点からある頂点へと移動する時にそれぞれの頂点を高々一回しか通らないような行き方のこと。

✳︎連結グラフ
無向グラフにおいてグラフの任意の二頂点間に道が存在するグラフのこと。

✳︎連結成分
あるグラフのうち極大で連結な部分グラフのこと。UnionFind木において分割されるグループと捉えるとわかりやすい。

✳︎強連結
有向グラフぶおいてグラフの任意の二頂点間に道が存在するグラフのこと。

✳︎強連結成分
あるグラフのうち極大で強連結な部分グラフのこと。UnionFind木の代わりに強連結成分分解というアルゴリズムが存在する。