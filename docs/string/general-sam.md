其实广义后缀自动机也没有什么特别多的内容。

### 前置

首先需要理解$SAM$的基本性质，对于一个字符串$S$，我们之所以把有向图$G$叫做$SAM$，其满足以下最重要的性质

1. 一个$S$的子串对应了一条从源点$t_0$出发的路径，一个从源点出发的路径唯一对应了一个$S$的子串。
2. 然后对于一个点$v$，唯一对应了一个$endpos$集合，也就是说所有从$t_0$到$v$的路径对应的字符串，$endpos$集合相同，字符串中一个$endpos$集合也唯一对应了一个点。

至于后缀链接，$len$之类的东西，都是一些辅助变量，上诉才是$SAM$的核心。

### 构造

类似地可以给出广义$SAM$的重要性质，对于一棵$trie$树$T$，把上诉$S$的子串更改为$T$的一条直链（也就是一条到根路径对应的字符串的子串），然后可以说，几乎$SAM$的所有性质，广义$SAM$都具有。

然后考虑构造$T$的广义$SAM$，其实无论是$SAM$还是广义$SAM$，只要你的构造方法满足上诉的一些性质，基本上就是对的了，我们只需要在构造$SAM$基础上做一些修改。我们需要清楚$SAM$的$extend$并不是加入一个字符，他的意思加入若干个后缀，同样对于广义$SAM$，我们考虑我们按照某种顺序加入每个$T$上的点，当然需要保证每次加入都只存在一个联通块，假设加入了点$x$，那么找到$x$的父亲$y$，设边$(y,x)$对应字符是$c$，找到$y$在广义$SAM$上对应的节点$z$，如果发现$z$没有到$c$的出边，就相当于是加入若干个 $x$到根路径的对应字符串 的后缀，显然可以套用$SAM$的过程，否则存在这样一个出边，假设指向$p$，那么如果$(z,p)$是连续的，即$len(p)=len(z)+1$，实际上需要加入的后缀都存在了，啥事不做即可。否则是不连续的，虽然加入的后缀都存在了，但是没有正确维护$endpos$和点之间的关系，我们需要将$p$拆出一个点$clone$，复制$p$点所有"信息"，让$len(clone)=len(z)+1$，然后从$z$跳后缀链接，把指向$p$的转移改到指向$clone$，其实类似$SAM$中拆点的做法。

如果把加入的顺序用$dfs$的话，时间复杂度将是$O(\sum_{i=1}^n|dep_i|)$(其中$dep_i$为$T$上点$$i$的深度)，如果使用$bfs$的话，时间复杂度是$O(|T|)$，证明是不可能会了，这辈子都不可能会了。

### [模板](https://www.luogu.com.cn/problem/P6139)

```cpp
#include <iostream>
#include <cstdio>
#define il inline
#define ri register
#define ll long long
#define Size 2000050
using namespace std;
char s[Size];
il void read(int&);
struct sam{
	struct state{
		int tran[26],link,len;
	}s[Size];
	int tot=1;
	il void extend(int&last,int c){
		int cur,p(last);
		if(s[last].tran[c])
			cur=0,last=s[s[last].tran[c]].len
				==s[last].len+1?s[last].tran[c]:tot+1;
		else s[cur=++tot].len=s[last].len+1,last=cur;
		while(p&&!s[p].tran[c])
			s[p].tran[c]=cur,p=s[p].link;
		if(p){int q(s[p].tran[c]);
			if(s[q].len==s[p].len+1)s[cur].link=q;
			else{
				int clone(++tot);
				s[clone]=s[q],s[clone].len=s[p].len+1;
				s[q].link=s[cur].link=clone;
				while(s[p].tran[c]==q)
					s[p].tran[c]=clone,
						p=s[p].link;
			}
		}else s[cur].link=1;
	}il void init(){
		ll ans(0);
		for(int i(2);i<=tot;++i)
			ans+=s[i].len-s[s[i].link].len;
		printf("%lld",ans);
	}
}S;
int main(){
	int n;read(n);
	for(int i(1),j,p;i<=n;++i){
		scanf("%s",s);
		for(j=0,p=1;s[j];++j)
			S.extend(p,s[j]-97);
	}return S.init(),0;
}
il void read(int&x){
	x^=x;ri char c;while(c=getchar(),c<'0'||c>'9');
	while(c>='0'&&c<='9')x=(x<<1)+(x<<3)+(c^48),c=getchar();
}

```
