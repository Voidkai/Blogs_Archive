> 具体算法分析见：[Practical Techniques for Searches on Encrypted Data][1]
## 算法一：

```Python
import nltk
from Crypto.Hash import HMAC, SHA256,SHA
import numpy as np

def stringXor(s1,s2):
    l = max(len(s1),len(s2))
    xor = str(hex(int(s1,16) ^ int(s2,16))).replace("0x","")
    dif = l - len(xor)
    add = ""
    while(dif != 0):
        add = add + "0"
        dif= dif-1

    return add+xor



def hmac(str,secret):
    m = HMAC.new(secret.encode("utf8"), digestmod=SHA256)
    m.update(str.encode("utf8"))
    return m.hexdigest()


def prf(str,secret):
    m = HMAC.new(secret.encode("utf8"), digestmod=SHA)
    m.update(str.encode("utf8"))
    return m.hexdigest()


def prF(str,secret):
    m = HMAC.new(secret.encode("utf8"))
    m.update(str.encode("utf8"))
    return m.hexdigest()


#获取随机数序列X
def getX(lenth,seed):
    np.random.seed(seed)
    X = []
    i = 0
    while i < lenth:
        tmp = []
        for b in range(0,32):
            tmp.append(str(hex(np.random.randint(1, 16))).replace("0x", ""))
        X.append("".join(tmp))
        i = i+1
    print("随机序列X")
    print(X)

    return X


def PBT(path:str):
    E = [] #二进制后
    F = []
    D = []  # 文本散列值
    K = []  # 密钥集
    k_s = 1234
    k_h = 'secret'
    k_r = 'PRF'

    file_object = open(path,'r')
    file_content = file_object.read()
    text = nltk.word_tokenize(file_content)
    print(text)

    X = getX(len(text), k_s)

    #for word in text:
    #    E.append(encode(word))
    #print(E)
    E = text

    for word in E:
        D.append(hmac(word, k_h))
    print("散列值D")
    print(D)
    for word in D:
        K.append(prf(word, k_r))
    print("密钥集K")
    print(K)

    for i in range(0, len(K)):
        F.append(prF(X[i], K[i]))
    print("F")
    print(F)

    #XOR得令牌序列
    T = []
    for i in range(0, len(D)):
        T.append(stringXor(D[i], X[i]+F[i]))
    print("令牌序列T")
    print(T)

    return T

tokencollection = PBT("Algorithm/input.txt") #input the plaintext blocks W
tokencollection = np.array(tokencollection)
np.save("Algorithm/TokenCollection.npy",tokencollection)
```



## 算法二：

```Python
import nltk
from Crypto.Hash import HMAC, SHA256,SHA
import numpy as np


def hmac(str,secret):
    m = HMAC.new(secret.encode("utf8"), digestmod=SHA256)
    m.update(str.encode("utf8"))
    return m.hexdigest()

def prf(str,secret):
    m = HMAC.new(secret.encode("utf8"), digestmod=SHA)
    m.update(str.encode("utf8"))
    return m.hexdigest()


def prF(str,secret):
    m = HMAC.new(secret.encode("utf8"))
    m.update(str.encode("utf8"))
    return m.hexdigest()


def readTxtLine(rootdir):
    lines = []
    with open(rootdir, 'r') as file_to_read:
        while True:
            line = file_to_read.readline()
            if not line:
                break
            line = line.strip('\n')
            lines.append(line)
    return lines


def RP(path: str):  # Rules Processing
    E = []  # 二进制后
    P = []  # 文本散列值
    K = []  # 密钥集
    k_h = 'secret'
    k_r = 'PRF'

    # 分行读取
    text = readTxtLine(path)

    for word in text:
        E.append(word)
    # print(E)

    for word in E:
        P.append(hmac(word, k_h))
    print("散列值P")
    print(P)
    for word in P:
        K.append(prf(word, k_r))
    print("密钥集K")
    print(K)

    return P, K


hashvalue, keyset = RP("rulecollection.txt")
np.save("hashvalueP.npy",np.array(hashvalue))
np.save("keysetK_rule.npy",np.array(keyset))
```



## 算法三：

```Python
from Crypto.Hash import HMAC, SHA256,SHA
import numpy as np
import time

def stringXor(s1,s2):
    l = max(len(s1), len(s2))
    xor = str(hex(int(s1,16) ^ int(s2,16))).replace("0x","")
    dif = l - len(xor)
    add = ""
    while(dif != 0):
        add = add + "0"
        dif= dif-1

    return add+xor


def hmac(str,secret):
    m = HMAC.new(secret.encode("utf8"), digestmod=SHA256)
    m.update(str.encode("utf8"))
    return m.hexdigest()

def prf(str,secret):
    m = HMAC.new(secret.encode("utf8"), digestmod=SHA)
    m.update(str.encode("utf8"))
    return m.hexdigest()


def prF(str,secret):
    m = HMAC.new(secret.encode("utf8"))
    m.update(str.encode("utf8"))
    return m.hexdigest()


def De(path_token,path_rulehash,path_keyset): #Detection
    T = np.load(path_token).tolist()
    P = np.load(path_rulehash).tolist()
    K= np.load(path_keyset).tolist()

    n_token = len(T)
    n_rule = len(P)
    m = 32
    for i in range(0, len(P)):
        U = []
        print("检验是否符合第" + str(i+1)+"个规则")
        for j in range(0,len(T)):
            U.append(stringXor(P[i],T[j]))

        F_2 = []  # 后m位
        F_1 = []  # 前n-m位

        position = []
        num = 0
        for n in range(0, len(U)):
            temp1 = str(prF(U[n][0:32], K[i]))
            temp2 = U[n][(len(U[0]) - m):len(U[0])]
            F_1.append(temp1)
            F_2.append(temp2)
            if(temp1 == temp2):
                position.append(n+1)
                num = num +1

        # print("F_1:", end='')
        # print(F_1)
        # print("F_2:", end='')
        # print(F_2)
        if(position == []):
            print("未发现匹配")
        else:
            #print("第"+str(position+1)+"个位置符合该规则")
            print("第", end='')
            print(*position,sep=",", end='')
            print("个位置匹配,共"+str(num)+"处匹配")

    return n_token, n_rule
time_start=time.time()

n1,n2=De("TokenCollection.npy","hashvalueP.npy","keysetK_rule.npy")

time_end=time.time()
print("在" + str(n1)+"个词的加密文本中" +"检测"+str(n2)+"条规则所用时间", end='' )
print(time_end-time_start,'s')
```


  [1]: https://kaixuan.site/index.php/archives/5/