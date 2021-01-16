---
title: Optimization by Julia, 1 
---


Julia

参考文献: 

1. Changhyun Kwon, Julia Programming for Operations Research, 2nd Edition, https://www.chkwon.net/julia/

2. https://jump.dev/JuMP.jl/v0.21.1/quickstart/



この書籍で用いられているコードは，上記のwebsiteから入手可能である。テストのために用いられているのは，Julia v1.3.0, JuMP v0.21.2, Optim v0.20.6である。

# 実行環境


ここでは，Visual Studio Code に 拡張機能としてJulia Languate Supportを導入し，実行する。


@Mizuto_Kadowaki,2020年12月15日に更新,「VSCode で Julia-1.4 を動かすまで」
https://qiita.com/Mizuto_Kadowaki/items/b95e4b7db4a1dfb59863


Juliaでは，最初の要素はv(1)と，添字1でアクセスされる。

Juliaで数理最適化を扱うためのパッケージとして，JuMPがある。 JuMPを用いると，数理最適化問題を簡単に記述することができる。 JuMPのように，人間が簡単に数理最適化問題を記述することができる言語を，モデリング言語という。

JuliaにJuMPをインストールするには，次のようにする。
```julia:
julia>import Pkg
julia>Pkg.add("JuMP")
```


例えば，

$$
\min \sum_{(i,j) \in A} c_{ij} x_{ij}
$$

をJuMPを用いて記述すると，


```julia:
@variable(m, 0<=x[links]<=1)
@objective(m,Min,sum(c[(i,j)]*x[(i,j)] for (i,j) in links))
```

のように記述することができる。

JuMPは数理最適化問題を記述するだけである。記述した問題を解くには，別にソルバーが必要である。解きたい問題のクラスに対応したソルバーを，Juliaにインストールする必要がある。ここでは，混合整数計画問題を解くためのソルバーとして，Cbcをインストールする。

```julia:
import Pkg
Pkg.add("Cbc")
```

```julia:
using JuMP
using Cbc
```


最適化問題を定義するには，`Model()`を用いてモデルを定める。
その際に，用いるソルバーを引数に指定することができる。

```julia:
julia>model = Model(Cbc.Optimizer)
```
これを実行すると，次のように表示される。
```julia:
A JuMP Model
Feasibility problem with:
Variables: 0
Model mode: AUTOMATIC
CachingOptimizer state: EMPTY_OPTIMIZER
Solver name: COIN Branch-and-Cut (Cbc)
```

このモデルに変数を追加するには，次のようにする。

```julia
julia @variable(model,0<=x<=2)
```
これは，0以上2以下の変数`x`を，`model`に追加するものである。
加えて，変数`y`も追加する。
```julia
julia @variable(model,0<=y<=30)
```
これらの変数を用いて制約式を追加するには，`@constraint`を用いて次のようにする。
```julia
julia>@constraint(model,con,1x+5y<=3)
```
これは，$x+5y\leq 3$という制約式を追加するものである。
ここで特徴的なのは，変数の係数を，数値と変数を並べて$5y$などと書くことである。他の言語では，`5*x`などと，積を表す記号を入れることが多いが，Juliaでは必要ない。ただし，`*`を入れた方がわかりやすいのであれば，入れてもよい。

目的関数を定義するには，`@objective`を用いて次のようにする。

```julia
julia>@objective(model,Max,5x+3*y)
```
ここで，2番目の引数`Max`は，目的が最大化であることを表す。

こうして定義した最適化問題を解くには，`optimze`を用いる。

```julia
julia>optimize!(model)
```
これを実行すると，次のように表示される。

```julia
Welcome to the CBC MILP Solver 
Version: 2.10.3 
Build Date: Nov  9 2020 

command line - Cbc_C_Interface -solve -quit (default strategy 1)
Presolve 1 (0) rows, 2 (0) columns and 2 (0) elements
0  Obj -0 Dual inf 28 (2)
1  Obj 15
Optimal - objective value 15
Optimal objective 15 - 1 iterations time 0.002
Total time (CPU seconds):       0.00   (Wallclock seconds):       0.00
```
この問題は線形計画問題であるが，解いた後の状態は，JuMPの関数である`JuMP.termination_status`で確認することができる。
```julia
julia>termination_status(model)
```
これを実行すると，`OPTIMAL:TerminationStatusCode = 1`と表示される。このことから，最適解が得られたことがわかる。

得られた最適値は，

```julia
julia>objective_value(model)
```
より15であることがわかる。
また，最適解は，
```julia
julia>value(x)
julia>value(y)
```
より，$(x,y)=(3,0)$であることがわかる。






