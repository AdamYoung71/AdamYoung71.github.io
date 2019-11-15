---
layout:	post
title:  "Survey On FHE"
date: 2019-11-7 12:40
author: "Adam"
header-img: "img/post-bg-Cryptography.jpg"
catalog: true
tags:
    - Cryptography
    - Homomorphic Encryption
---

# Survey On FHE (Schema & Application)

- References

  > "Fully Homomorphic Encryption over the Integers" --Marten van Dijk et al.
  >
  > "Computing Arbitrary Functions of Encrypted Data" --Gentry

### 1. FHE Schema

- **A simple function with homomorphic characteristic:**

  > Modular Operation here: a mod p = a -[a/p] * p where [x] is the closest integer to x, so that the range of a mod p is now {-p/2, p/2}.

  $c=Enc(m)=m+pq+2r$

  $m=Dec(c)=(c\ mod\ p)mod\ 2$

  Plaintext Space: $m\in \{0,1\}​$ (operates on bits)

  Ciphertext Space : $N$ (cipher can be any integer)

  Key: $P$, can be used as private key where public key be $pq$.

- **Correctness**

  $Dec(c_i+c_j)=Dec((m_i+m_j))+p(q_i+q_j)+2(r_i+r_j))=((m_i+m_j)+2(r_i+r_j))mod(2)=m_i+m_j​$

  $Dec(c_i*c_j)=(m_i+2r_i)(m_2+2r_2)mod(2)=m_1*m_2​$

### 2. Noise

As long as the Public Key is $pq​$, we can subtract it from the ciphertext and get :

$c-pq=m+2r​$

Because of the interference caused by $2r​$, we could not inference the plaintext. So $m+2r​$ is recognized as "Noise", and we can know that it augments along with the operation on ciphertext. Note that when $m+2r​$ is bigger than $\frac{p}{2}​$ in absolute value，$c\ mod(p)​$ will not produce prospective result, making it only available on low-polynomial-order functions, so named as SWHE. 



### 3. Re-encryption

-  A direct way of noise-cancelling is to decrypt the ciphertext, while it is not quite possible to acquire the private key, so what if we encrypt the key? 

- **Premises:** 

  ​	- Evaluate: the algorithm$\epsilon(pk,f,c_i)$ outputs a ciphertext that encrypts $f(c_i)$

  ​	- Permitted Function:such functions satisfies  Evaluate.

  ​	- Fresh Ciphertext: Ciphertext encrypted only once. 

- **Re-encryption**: if $Dec$ is also a *permitted Function*, we can calculate: 

  $\epsilon (pk_2, Dec, s^*,c^*)$

  where $s^*=Enc(pk_2,pk)$ and $c^*=Enc(pk2, c)$

  Now we did a decryption on the ciphertext, making it another *fresh ciphertext*, thus mitigate the overall noise. This approach is called "Homomorphic Re-Decryption", it works like changing someone's outfit without taking off any of the original clothes. 


