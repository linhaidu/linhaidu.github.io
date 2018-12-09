---
title: "算法系列之Scrypt算法"
excerpt: 介绍Scrypt算法，以及litecoin的优化

categories:
  - Algorithm
tags:
  - Original


---

# 1 Scrypt算法 

Scrypt算法

```c++
Input:
P       Passphrase, an octet string.
S       Salt, an octet string.
N       CPU/Memory cost parameter, must be larger than 1, 
        a power of 2, and less than 2^(128 * r / 8).
r       Block size parameter.
p       Parallelization parameter, a positive integer
        less than or equal to ((2^32-1) * hLen) / MFLen
        where hLen is 32 and MFlen is 128 * r.
dkLen   Intended output length in octets of the derived
        key; a positive integer less than or equal to
        (2^32 - 1) * hLen where hLen is 32.        
Output:
DK      Derived key, of length dkLen octets.
```



Scrypt计算步骤

```c++
1. Initialize an array B consisting of p blocks of 
   128 * r octets each: B[0] || B[1] || ... || B[p - 1] 
   = PBKDF2-HMAC-SHA256 (P, S, 1, p * 128 * r)
2. for i = 0 to p - 1 do
       B[i] = scryptROMix (r, B[i], N)
3. DK = PBKDF2-HMAC-SHA256 (P, B[0] ||...|| B[p - 1],1,dkLen)
```

主要包含两个算法，PBKDF2-HMAC-SHA256，scryptROMix，下面分别介绍。

## 1.1 PBKDF2-HMAC-SHA-256 

PBKDF2-HMAC-SHA-256 是PBKDF2算法 [RFC2898] 采用HMAC-SHA-256 [RFC6234] 作为伪随机函数(PRF)。HMAC-SHA-256函数输出是32字节 。

### 1.1.1 PBKDF2

PBKDF2算法

```c++
Options:   PRF        underlying pseudorandom function (hLen
                      denotes the length in octets of the
                      pseudorandom function output)
Input:     P          password, an octet string
           S          salt, an octet string
           c          iteration count, a positive integer
           dkLen      intended length in octets of the derived
                      key, a positive integer, at most
                      (2^32 - 1) * hLen
Output:    DK         derived key, a dkLen-octet string
```



PBKDF2计算步骤

```c++
1. If dkLen > (2^32 - 1) * hLen, output "derived key too 
    long" and stop.
2. Let l be the number of hLen-octet blocks in the derived 
   key, rounding up, and let r be the number of octets in 
   the last block:
   l = CEIL (dkLen / hLen) 
   r = dkLen - (l - 1) * hLen 
   Here, CEIL (x) is the "ceiling" function, i.e. the 
   smallest integer greater than, or equal to, x. 
3. For each block of the derived key apply the function F 
   defined below to the password P,the salt S, the iteration 
   count c, and the block index to compute the block:
   T_1 = F(P, S, c, 1) 
   T_2 = F(P, S, c, 2) 
   ...
   T_l = F(P, S, c, l) 
   where the function F is defined as the exclusive-or sum of 
   the first c iterates of the underlying pseudorandom 
   function PRF applied to the password P and the 
   concatenation of the salt S and the block index i:
   F(P, S, c, i) = U_1 \xor U_2 \xor ... \xor U_c
   U_1 = PRF (P, S || INT (i))
   U_2 = PRF (P, U_1)
   ...
   U_c = PRF (P, U_{c-1})
   Here, INT (i) is a four-octet encoding of the integer i, 
   most significant octet first.
4. Concatenate the blocks and extract the first dkLen octets 
   to produce a derived key DK:
   DK = T_1 || T_2 ||  ...  || T_l<0..r-1>
5. Output the derived key DK.
```

### 1.1.2 HMAC

HMAC算法：

```c++
H(K XOR opad, H(K XOR ipad, text))
where:
ipad = the byte 0x36 repeated B times
opad = the byte 0x5C repeated B times
```



HMAC计算步骤：

```c++
1. append zeros to the end of K to create a B byte string 
   (e.g., if K is of length 20 bytes and B=64, then K will 
   be appended with 44 zero bytes 0x00). The key for HMAC 
   can be of any length (keys longer than B bytes are first
   hashed using H)
2. XOR (bitwise exclusive-OR) the B byte string computed 
   in step 1 with ipad
3. append the stream of data ’text’ to the B byte string 
   resulting from step 2
4. apply H to the stream generated in step 3
5. XOR (bitwise exclusive-OR) the B byte string computed 
   in step 1 with opad
6. append the H result from step 4 to the B byte string 
   resulting from step 5
7. apply H to the stream generated in step 6 and output the 
   result
```

### 1.1.3 SHA256

HMAC中的H就是sha256



## 1.2 scryptROMix

scryptROMix算法

```c++
Input:
r       Block size parameter.
B       Input octet vector of length 128 * r octets.
N       CPU/Memory cost parameter, must be larger than 1,
        a power of 2, and less than 2^(128 * r / 8).
Output:
B’      Output octet vector of length 128 * r octets.
```

scryptROMix计算步骤

```c++
1. X = B
2. for i = 0 to N - 1 do
       V[i] = X
       X = scryptBlockMix (X)
     end for
3. for i = 0 to N - 1 do
       j = Integerify (X) mod N
           where Integerify (B[0] ... B[2 * r - 1]) is 
           defined as the result of interpreting B[2 * r - 1] 
           as a little-endian integer.
       T = X xor V[j]
       X = scryptBlockMix (T)
     end for
4. B’ = X
```

### 1.2.1 scryptBlockMix 

scryptBlockMix算法

```c++
Input:
       B[0] || B[1] || ... || B[2 * r - 1]
       Input octet string (of size 128 * r octets),
       treated as 2 * r 64-octet blocks,
       where each element in B is a 64-octet block.
Output:
       B’[0] || B’[1] || ... || B’[2 * r - 1]
```

scryptBlockMix计算步骤

```c++
1. X = B[2 * r - 1]
2. for i = 0 to 2 * r - 1 do
     T = X xor B[i]
     X = Salsa (T)
     Y[i] = X
   end for
3. B’ = (Y[0], Y[2], ..., Y[2 * r - 2],
         Y[1], Y[3], ..., Y[2 * r - 1])
```

### 1.2.2 Salsa

Salsa算法

```c++
#define R(a,b) (((a) << (b)) | ((a) >> (32 - (b))))
void salsa20_word_specification(uint32 out[16],uint32 in[16])
   {
     int i;
     uint32 x[16];
     for (i = 0;i < 16;++i) x[i] = in[i];
     for (i = 8;i > 0;i -= 2) {
         x[ 4] ^= R(x[ 0]+x[12], 7);  
         x[ 8] ^= R(x[ 4]+x[ 0], 9);
         x[12] ^= R(x[ 8]+x[ 4],13);  
         x[ 0] ^= R(x[12]+x[ 8],18);
         x[ 9] ^= R(x[ 5]+x[ 1], 7);  
         x[13] ^= R(x[ 9]+x[ 5], 9);
         x[ 1] ^= R(x[13]+x[ 9],13);  
         x[ 5] ^= R(x[ 1]+x[13],18);
         x[14] ^= R(x[10]+x[ 6], 7);  
         x[ 2] ^= R(x[14]+x[10], 9);
         x[ 6] ^= R(x[ 2]+x[14],13);  
         x[10] ^= R(x[ 6]+x[ 2],18);
         x[ 3] ^= R(x[15]+x[11], 7);  
         x[ 7] ^= R(x[ 3]+x[15], 9);
         x[11] ^= R(x[ 7]+x[ 3],13);  
         x[15] ^= R(x[11]+x[ 7],18);
         
         x[ 1] ^= R(x[ 0]+x[ 3], 7);  
         x[ 2] ^= R(x[ 1]+x[ 0], 9);
         x[ 3] ^= R(x[ 2]+x[ 1],13);  
         x[ 0] ^= R(x[ 3]+x[ 2],18);
         x[ 6] ^= R(x[ 5]+x[ 4], 7);  
         x[ 7] ^= R(x[ 6]+x[ 5], 9);
         x[ 4] ^= R(x[ 7]+x[ 6],13);  
         x[ 5] ^= R(x[ 4]+x[ 7],18);
         x[11] ^= R(x[10]+x[ 9], 7);  
         x[ 8] ^= R(x[11]+x[10], 9);
         x[ 9] ^= R(x[ 8]+x[11],13);  
         x[10] ^= R(x[ 9]+x[ 8],18);
         x[12] ^= R(x[15]+x[14], 7);  
         x[13] ^= R(x[12]+x[15], 9);
         x[14] ^= R(x[13]+x[12],13);  
         x[15] ^= R(x[14]+x[13],18);
}
     for (i = 0;i < 16;++i) out[i] = x[i] + in[i];
   }
```



# 2 Litecoin

litecoin采用Scrypt算法，并且P=S，N=1024，r=p=1，dkLen=32。

## 2.1 Litecoin算法

Scrypt算法简化成如下

```c++
1. B[0] = PBKDF2-HMAC-SHA256 (P, S, 1, 128)
2. B[0] = scryptROMix (1, B[0], 1024)
3. DK = PBKDF2-HMAC-SHA256 (P, B[0],1, 32)
```

PBKDF2算法简化如下

```c++
1. l = CEIL (dkLen / hLen) = 128/32 = 4
   r = dkLen - (l - 1) * hLen = 0   
2. T_1 = F(P, S, c, 1) = U_1 = PRF (P, S || INT (1))
   T_2 = F(P, S, c, 2) = U_2 = PRF (P, S || INT (2))
   T_3 = F(P, S, c, 3) = U_3 = PRF (P, S || INT (3))
   T_4 = F(P, S, c, 4) = U_4 = PRF (P, S || INT (4))  
3. DK = T_1 || T_2 ||  T_3  || T_4
```

scryptROMix计算步骤

```c++
1. X = B
2. for i = 0 to N - 1 do
       V[i] = X
       X = scryptBlockMix (X)
     end for
3. for i = 0 to N - 1 do
       j = Integerify (X) mod N
           where Integerify (B[0] ... B[2 * r - 1]) is 
           defined as the result of interpreting B[2 * r - 1] 
           as a little-endian integer.
       T = X xor V[j]
       X = scryptBlockMix (T)
     end for
4. B’ = X
```

scryptBlockMix算法简化如下

```c++
1. X = B[1]
2. for i = 0 to 1 do
     T = X xor B[i]
     X = Salsa (T)
     Y[i] = X
   end for
3. B’ = (Y[0], Y[1])
```

## 2.2 Litecoin算法优化

略
