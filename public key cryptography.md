We have shown that given generator point G, and select a scalar e (big enough), we can compute P=e*G easily, but when you have G and P, its impossible to get e. 
This is the basic for elliptic curve graphy.

The scalar k we selected is called private key, and Q is public key, notice that k is a 256 bits integer and Q contains two part one is the x coordinate and the other
is y coordinate. 

The public key cryptography has its name as Elliptic Curve Digital Signature Algorithm, known as ECDSA for short. It invovles following steps:

1, select a scalar e, and compute P = e*G, e is private key and P is public key, realse P to public and every one can know it.

2, the owner of private key randomly select two finite field member u, v, and compute k = u + ve , k need to keep secret.

3, compute R = k * G = (u + v*e)*G = u*G + v*(e*G) = u*G+v*P, we only use the x coordinate of R, and name the value of x as r

4, the owner of private key generate a given message in text format with any length(it can be published), and hash it to a 256 bits number by using sha256 or md5, 
name this hash result as z，

5, compute a number s by s = (z + r*e) / k (all these computation are base on modulur p)

6, realse the turple (z, s，r) as the signiture of the private key owner

7, any one who want to make sure the message is really created by the owner of the private key, he or she can do the following steps:
   1, compute u = z / s, v = r / s,
   2, compute u* G + v * P = (z/s)*G + (r/s)*P = (z/s)*G + (r/s)*(e*G) = (z/s)*G + (re/s)*G = ((z+re)/s)*G = k*G = R',
   take the x coordinate of R' and check it with r, if they are the same, then we can be sure that message z is really created by the owner of the private key

Let's put these steps into code as following:
```g
```
