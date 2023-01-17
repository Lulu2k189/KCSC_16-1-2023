# secRet Spy x fAmily
## Decription
Nay Forger Loid trở về nhà với một khuôn mặt đầy sự mệt mỏi. Anya đọc suy nghĩ của Papa và phát hiện Papa đang đau đầu để giải mã bức thông điệp then chốt cho hòa bình 2 miền Đông Tây. Hãy cùng Anya giải mã bức thông điệp!
![anya](https://user-images.githubusercontent.com/122846300/212797718-e3d1e517-49ac-412f-ae93-18b74ca06091.png)

[output.txt](https://github.com/Lulu2k189/KCSC_16-1-2023/files/10430638/output.txt)

source.py
```python
from Crypto.Util.number import *
from secret import flag
e = 65537
flag = b'KCSC{XXXXXXXXXXXX}'
def reverse(n):{
    #function to reverse binary of n
}

while True:
    p = getPrime(1024)
    q = inverse(e, p)
    if not isPrime(q):
        continue
    n = p * q
    cipher = pow(bytes_to_long(flag),e,n)
    n = reverse(n) 
    f = open('output.txt','w')
    print(f"N = {n}",file=f)
    print(f"e = {e}",file=f)
    print(f"Cipher = {cipher}",file=f)
```
## Cách giải
Thuật toán RSA không còn xa lạ với các bạn sinh viên trường mình. Đây là 1 bài thuần túy tìm flag bị mã hóa thông qua hệ mật RSA.

Trước khi đi đến những bước cao hơn, phải đi được những bước thấp đã.

```python
n = reverse(n)
```
Ngay trước khi in giá trị n vào file txt, mình có đưa nó qua 1 hàm reverse như sau:
```python
def reverse(n):{
    #function to reverse binary of n
}
```
Hàm này có tác dụng đảo ngược giá trị bin của n, VD: n = 0b10111011 => 0b11011101.

Để đảo ngược lại, các bạn có thể dùng một code đơn giản như sau:

```python
n = int(bin(N)[2:][::-1],2)
```
Giờ chúng ta đã có modulus n chính xác, đã đến lúc tìm flag.

Việc đầu tiên mà các bạn phải làm khi gặp một bài RSA cho n, đó là đưa lên https://factordb.com/ để xem có thể factor n ra p và q hay không.

Tất nhiên là bài này không thể làm thế :)))))

Chúng ta xem xét code tạo p và q:

```python
 p = getPrime(1024)
 q = inverse(e, p)
 if not isPrime(q):
    continue
```
getPrime dùng để tạo số nguyên tố 1024bit p, sau đó q là nghịch đảo của e qua modulus p.

Có thể hình dung mối quan hệ của p và q như sau:

```python
q = e**(-1) mod p
```
Dựa vào công thức trên, ta có:
```python
q * e = 1 + k * p với k ∈ N (1)
```
Lại có:
```python
n = p * q              
n * e = p * q * e           (2)
```
Từ (1) và (2) :
```python
n * e = p * (1 + k * p)
k * (p ** 2) + p - n * e = 0   với k ∈ N  (3)
```
Đây là phương trình bậc 2 ẩn p. Đến đây ta cần bruteforce k để tìm p.

Nhắc lại về phương trình bậc 2: phương trình bậc 2 có nghiệm khi và chỉ khi delta = b**2 - 4ac >= 0. 

Như với phương trình (3) giá trị của delta là:
```python
delta = 1 + 4kne
```
Về cơ bản delta không thể bằng 0 vì n và e là số dương đã biết, k cũng là số nguyên. Như vậy chỉ có trường hợp delta > 0.

Các bạn để ý là với delta > 0, luôn có 1 nghiệm âm và 1 nghiệm dương. Số p của ta không thể là số âm , nên ta chỉ xét nghiệm dương của phương trình. Nghiệm đó có dạng: x1 = (-b + sqrt(delta))  / 2a

![image](https://user-images.githubusercontent.com/122846300/212804142-637d30cb-397d-44ba-8830-06e7a771127b.png)

Sử dụng iroot của sympy, có thể dễ dàng tìm ra nghiệm:
```python
from gmpy2 import iroot
delta = 1 + 4*k*n*e
sqr = iroot(delta,2)[0]
```
Đến đây bruteforce được rồi đấy :)

solve.py
```python
from gmpy2 import iroot
from Crypto.Util.number import *

N = 13524733362044232010736349595937464796777752381282044858904535549598695415259485540306280531287404403541039855661018351433654227342950710134306170544748441173774593898898768923907735212448495520658645896437678387670188401560165044043577269969416130117140222729708904953465161496726313677920240975640999580444678641072701612002184731572732086781180661428229265121878536891979130796853074814897843058889565035037987946761236280609366824566706376823708457581218333217098354233306312287549927586100725051273954453617494755601749327688591032955254157726764321151877093977335572150146339853526534429133266975640029987854181
e = 65537
Cipher = 3678620397255743852788853741107986108860981676309723331641730290544731009447106060307185476663548386172571801713658668783790148410569386488760778490944114348420614589196065297281576228195492339713201469195237426515519163981864121592862248581092535664172672153903696701157467812876812796524706730400133900257530235932000707810825149259236944828592989457788651053131965278448678769098840150302476398856999420129829083563611916558475165369663734941095660119099130765405955767410469621006246831917091632524062249541513097151767121519911807673852738866453379562168834013804186606786287157327927655505568604704262776577804

n = int(bin(N)[2:][::-1],2)

p = 0
q = 0
 
for k in range(2,100000):
    delta = 1 + 4*k*n*e
    sqr = iroot(delta,2)[0]
    p1 = (-1+sqr)//(2*k)
    q1 = n//p1
    if p1*q1 - n == 0:
        p = p1
        q = q1
        print(k)
phi = (p-1 )*(q-1)
d = pow(e,-1,phi)
m = long_to_bytes(pow(Cipher,d,n))
print(m)
```
>Flag: KCSC{1f_D4m14n_g3t_m4rr13d_w1th_4ny4,th3_34st_4nd_th3_W3st_w1ll_b3_p34c3ful!!!!}


