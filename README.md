# week_16
## 快乐前缀问题
**leetcode 1392 最长快乐前缀**
暴力解法不再满足时间性能要求。使用哈希的方式进行性能的优化。
先计算最长前缀和最长后缀的哈希值，分成从前往后和从后往前分成前缀和后缀字符串，依次减去最后一个字符或第一个字符后的两种字符串的哈希值，比较两种字符串哈希值是否相等，不等则继续各去掉一个字符计算并比较，直到相等或为空。
下面问题就是如何根据之前字符串哈希值计算去掉一个字符后的哈希值。但是求模运算对于除法不成立，无法实现从长字符串哈希值计算短字符串哈希值，可以存储每一个从短字符串到长字符串的哈希值，在使用时直接找。形成以空间换时间的方法。
```c++
class Solution {
public:
    long MOD=(long)(1e9+7);
    string longestPrefix(string s) {
        //计算(26^i)%MOD的值
         long pow26[s.length()];
         pow26[0]=1;
         for(int i=1;i<s.length();i++){
             pow26[i]=pow26[i-1]*26%MOD;
         }
        //prehash[i]:hash(s[1...i])字符串0-i的哈希值
        long prehash[s.length()];
        prehash[0]=s[0]-'a';
        for(int i=1;i<s.length();i++){
            prehash[i]=(prehash[i-1]*26+s[i]-'a')%MOD;
        }
        //posthash[i]:hash(s[i...s.length-1])字符串i-(length-1)的哈希值
        long posthash[s.length()];
        posthash[s.length()-1]=s[s.length()-1]-'a';
        for(int i=s.length()-2;i>=0;i--){
            posthash[i]=((s[i]-'a')*pow26[s.length()-i-1]+posthash[i+1])%MOD;
        }

        //比较前缀哈希值与后缀哈希值是否相等
        for(int l=s.length()-1;l>0;l--){
            if(prehash[l-1]==posthash[s.length()-l]&& equal(s,0,l-1,s.length()-l,s.length()-1))
                return s.substr(0,l);
        } 
        return ""; 
    }
    //s[l1,r1]?=s[l2,r2]
    bool equal(string s,int l1,int r1,int l2,int r2){
        for(int i=l1,j=l2;i<=r1&&j<=r2;i++,j++){
            if(s[i]!=s[j])return false;
        }
        return true;
    }
};
```
## 重复的DNA问题
**leetcode 187 重复的DNA序列**
使用哈希表的方式遍历字符串中所有长度为10的子串，存储重复的。
```c++
class Solution {
public:
    vector<string> findRepeatedDnaSequences(string s) {
        unordered_set<string> seen;
        unordered_set<string> res;
        for(int i=0;i+10<=s.length();i++){
            string key=s.substr(i,10);
            if(seen.count(key)==0)
            seen.insert(key);
            else
            res.insert(key);
        }
        vector<string> ans;
        for(auto it=res.begin();it!=res.end();++it){
            ans.push_back(*it);
        }
        return ans;
    }
};
```
### 滚动哈希求解
选取前九个字符作为字符串，每次加上第一个字符计算哈希值再减去一个字符计算哈希值，类似滚动。因为每个字符只有ACGT四种取值，可以使A->1,C->2,G->3,T->4，这样长度为10的字符串的哈希值可以用一个10位的十进制数字表示（相对简单，用4进制应该更好）。
```c++
class Solution {
public:
    vector<string> findRepeatedDnaSequences(string s) {
        vector<string> ans;
        if(s.length()<10) return ans;
        map<char,int>map;
        map['A']=1;
        map['C']=2;
        map['G']=3;
        map['T']=4;

        unordered_set<long> seen;
        unordered_set<string> res;
        long hash=0,ten9=(long)1e9;
        for(int i=0;i<9;i++)
        hash=hash*10+map[s[i]];

        for(int i=9;i<s.length();i++){
            hash=hash*10+map[s[i]];
            if(seen.count(hash)==0)
            seen.insert(hash);
            else
            res.insert(s.substr(i-9,10));

            hash-=map[s[i-9]]*ten9;
        }
       
        for(auto it=res.begin();it!=res.end();++it){
            ans.push_back(*it);
        }
        return ans;
    }
};
```
